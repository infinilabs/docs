---
title: "内存与缓存"
date: 0001-01-01
summary: "内存与缓存配置 #  本页介绍 easysearch.yml 中与内存锁定和缓存相关的配置项。
 bootstrap.memory_lock #  bootstrap.memory_lock: true    项目 说明     参数 bootstrap.memory_lock   默认值 false   属性 静态   说明 启动时锁定 JVM 堆内存，防止操作系统将堆内存交换（swap）到磁盘。生产环境强烈建议启用    为什么需要锁定内存？ #  当操作系统将 JVM 堆内存换出到磁盘时，会导致：
 GC 暂停时间显著增加：GC 需要将被换出的页面重新换入内存。 搜索和索引延迟飙升：毫秒级操作可能变成秒级。 节点被判定为不可用：主节点可能因为超时将该节点踢出集群。  配合设置 #  启用 memory_lock 需要操作系统级的配合：
1. 设置 ulimit（systemd）
# /etc/systemd/system/easysearch.service.d/override.conf [Service] LimitMEMLOCK=infinity 2. 设置 limits.conf
# /etc/security/limits."
---


# 内存与缓存配置

本页介绍 `easysearch.yml` 中与内存锁定和缓存相关的配置项。

---

## bootstrap.memory_lock

```yaml
bootstrap.memory_lock: true
```

| 项目 | 说明 |
|------|------|
| **参数** | `bootstrap.memory_lock` |
| **默认值** | `false` |
| **属性** | 静态 |
| **说明** | 启动时锁定 JVM 堆内存，防止操作系统将堆内存交换（swap）到磁盘。**生产环境强烈建议启用** |

### 为什么需要锁定内存？

当操作系统将 JVM 堆内存换出到磁盘时，会导致：

- **GC 暂停时间显著增加**：GC 需要将被换出的页面重新换入内存。
- **搜索和索引延迟飙升**：毫秒级操作可能变成秒级。
- **节点被判定为不可用**：主节点可能因为超时将该节点踢出集群。

### 配合设置

启用 `memory_lock` 需要操作系统级的配合：

**1. 设置 ulimit（systemd）**

```ini
# /etc/systemd/system/easysearch.service.d/override.conf
[Service]
LimitMEMLOCK=infinity
```

**2. 设置 limits.conf**

```bash
# /etc/security/limits.conf
easysearch  soft  memlock  unlimited
easysearch  hard  memlock  unlimited
```

**3. 验证是否生效**

```bash
GET _nodes?filter_path=**.mlockall
```

返回 `"mlockall": true` 表示内存锁定成功。

> 如果启动日志出现 `Unable to lock JVM Memory` 错误，说明操作系统限制未正确配置。详见 [系统调优]({{< relref "../settings.md" >}})。

---

## 缓存设置

Easysearch 使用多级缓存来加速查询。以下参数控制各类缓存的大小。

### indices.fielddata.cache.size

```yaml
indices.fielddata.cache.size: 40%
```

| 项目 | 说明 |
|------|------|
| **参数** | `indices.fielddata.cache.size` |
| **默认值** | 无限制（不设上限，按需加载） |
| **属性** | **动态**（可通过集群设置 API 修改） |
| **说明** | 字段数据（fielddata）缓存的上限。支持百分比（堆内存占比）或绝对值（如 `4gb`）。当缓存达到上限时，会触发逐出（eviction），可能导致查询变慢 |

**什么是 Fielddata？**

- Fielddata 是对 `text` 字段进行排序、聚合时加载到内存中的倒排索引的正排形式。
- 它非常消耗内存，通常建议使用 `keyword` 字段或 `doc_values` 代替。
- 如果确实需要使用 fielddata，建议设置缓存上限防止 OOM。

### indices.queries.cache.size

```yaml
indices.queries.cache.size: 10%
```

| 项目 | 说明 |
|------|------|
| **参数** | `indices.queries.cache.size` |
| **默认值** | 堆内存的 `10%` |
| **属性** | 动态（可通过集群设置 API 修改） |
| **说明** | 节点级别的查询缓存（Node Query Cache）大小。缓存 `filter` 上下文中的查询结果（bitset），对重复的过滤查询有显著加速效果 |

**缓存触发条件**：

- 查询必须在 `filter` 上下文中（如 `bool.filter`、`constant_score`）。
- 段（segment）中文档数超过 10,000 且占该分片总文档数的 3% 以上。

### indices.requests.cache.size

```yaml
indices.requests.cache.size: 1%
```

| 项目 | 说明 |
|------|------|
| **参数** | `indices.requests.cache.size` |
| **默认值** | 堆内存的 `1%` |
| **属性** | 动态（可通过集群设置 API 修改） |
| **说明** | 分片级别的请求缓存（Shard Request Cache）大小。缓存**搜索请求的最终结果**（聚合结果、`count`、`_search` 的 `size: 0` 等），索引刷新后自动失效 |

**适用场景**：

- 对变化不频繁的索引执行相同的聚合查询。
- 仪表盘（dashboard）场景，相同查询反复执行。

---

## 堆内存分配概览

理解各组件如何瓜分堆内存，有助于合理配置缓存：

```
JVM 堆内存（-Xmx）
├── 查询缓存（10%）              indices.queries.cache.size
├── 请求缓存（1%）               indices.requests.cache.size
├── Fielddata 缓存（无限制）     indices.fielddata.cache.size
├── 索引缓冲区（10%）            indices.memory.index_buffer_size
├── 断路器保护（95% 总上限）     indices.breaker.total.limit
└── 剩余：Lucene 段读取、聚合、排序等运行时开销
```

> 缓存参数的百分比值都是相对于 JVM 堆内存（`-Xmx`）的比例。
> `indices.breaker.total.limit` 默认值为 `95%`（当 `indices.breaker.total.use_real_memory=true` 时），或 `70%`（当设为 false 时）。

---

## 配置示例

### 生产环境（通用）

```yaml
bootstrap.memory_lock: true
# 缓存使用默认值即可，除非有特定性能问题
```

### 聚合密集型场景

```yaml
bootstrap.memory_lock: true
indices.fielddata.cache.size: 30%      # 限制 fielddata 防止 OOM
indices.requests.cache.size: 2%        # 增大请求缓存
```

### 写入密集型场景

```yaml
bootstrap.memory_lock: true
indices.queries.cache.size: 5%         # 减少查询缓存，留更多堆内存给写入缓冲
```

---

## 延伸阅读

- [JVM 配置]({{< relref "./jvm.md" >}}) — 堆大小设置
- [系统调优]({{< relref "../settings.md" >}}) — `memlock` 等操作系统参数
- [硬件配置]({{< relref "../hardware.md" >}}) — 内存规格选型
- [集群配置]({{< relref "../configuration.md" >}}) — 断路器等动态设置

