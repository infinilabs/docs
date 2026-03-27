---
title: "线程池与断路器"
date: 0001-01-01
summary: "线程池与断路器配置 #  本页介绍 easysearch.yml 中与线程池、断路器和 API 兼容相关的配置项。
 ⚠️ 警告：Easysearch 默认的线程池配置已经过充分调优。除非有明确的性能瓶颈证据，不建议修改线程池参数。盲目增大线程数反而可能导致性能下降。
  线程池 #  Easysearch 为不同类型的操作使用不同的线程池。
线程池类型 #     线程池 用途 默认大小 默认队列     search 搜索请求 CPU × 3 / 2 + 1（向下取整） 1000   write 索引/删除/更新/bulk 写入 CPU 核数 10000   index 单文档索引 CPU 核数 200   get GET 请求 CPU 核数 1000   analyze 分析请求 1 16   management 集群管理 5 —   flush Flush 操作 CPU / 2（向下取整，至少 1） —   refresh 刷新操作 CPU / 2（向下取整，至少 1） —   snapshot 快照/恢复 CPU / 2（向下取整，至少 1） —   warmer 预热操作 CPU / 2（向下取整，至少 1） —   force_merge 强制合并 1 —    配置语法 #  # 线程池大小 thread_pool."
---


# 线程池与断路器配置

本页介绍 `easysearch.yml` 中与线程池、断路器和 API 兼容相关的配置项。

> ⚠️ **警告**：Easysearch 默认的线程池配置已经过充分调优。除非有明确的性能瓶颈证据，**不建议修改线程池参数**。盲目增大线程数反而可能导致性能下降。

---

## 线程池

Easysearch 为不同类型的操作使用不同的线程池。

### 线程池类型

| 线程池 | 用途 | 默认大小 | 默认队列 |
|--------|------|----------|----------|
| `search` | 搜索请求 | CPU × 3 / 2 + 1（向下取整） | 1000 |
| `write` | 索引/删除/更新/bulk 写入 | CPU 核数 | 10000 |
| `index` | 单文档索引 | CPU 核数 | 200 |
| `get` | GET 请求 | CPU 核数 | 1000 |
| `analyze` | 分析请求 | 1 | 16 |
| `management` | 集群管理 | 5 | — |
| `flush` | Flush 操作 | CPU / 2（向下取整，至少 1） | — |
| `refresh` | 刷新操作 | CPU / 2（向下取整，至少 1） | — |
| `snapshot` | 快照/恢复 | CPU / 2（向下取整，至少 1） | — |
| `warmer` | 预热操作 | CPU / 2（向下取整，至少 1） | — |
| `force_merge` | 强制合并 | 1 | — |

### 配置语法

```yaml
# 线程池大小
thread_pool.<pool_name>.size: <number>

# 队列大小（-1 表示无限制，不推荐）
thread_pool.<pool_name>.queue_size: <number>
```

### 配置示例

```yaml
# 搜索线程池（仅在搜索负载极高时调整）
thread_pool.search.size: 25
thread_pool.search.queue_size: 1000

# 写入线程池（仅在写入负载极高时调整）
thread_pool.write.size: 16
thread_pool.write.queue_size: 10000

# 索引线程池
thread_pool.index.size: 16
thread_pool.index.queue_size: 200
```

### 调优原则

1. **不要随意增大线程数**。线程过多会导致上下文切换开销增大、竞争加剧，反而降低性能。
2. **关注队列拒绝率**。通过 `GET _cat/thread_pool?v` 观察 `rejected` 列。如果某个线程池频繁拒绝，说明该类型操作的负载过高。
3. **先排查根本原因**。线程池拒绝通常是其他问题的表象（如慢查询、磁盘 I/O 瓶颈、GC 暂停等）。
4. **增大队列不如增加节点**。队列只是临时缓冲，长期队列积压会增加延迟和 OOM 风险。

### 监控线程池

```bash
# 查看所有线程池状态
GET _cat/thread_pool?v&h=name,active,queue,rejected,completed

# 查看特定线程池
GET _cat/thread_pool/search?v

# 节点级别的线程池统计
GET _nodes/stats/thread_pool
```

---

## 断路器

断路器防止操作消耗过多内存导致 OutOfMemoryError。当请求预估将超过内存限制时，断路器会主动拒绝请求并返回错误，而不是让 JVM 崩溃。

> 断路器参数为**动态设置**，可通过 [集群配置 API]({{< relref "../configuration.md" >}}) 在线修改。这里仅做参考说明。

### 断路器参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `indices.breaker.total.limit` | `95%` | 所有断路器的聚合限制（启用真实内存断路器时）。所有断路器的内存使用之和不得超过此值 |
| `indices.breaker.fielddata.limit` | `40%` | Fielddata 断路器。加载 fielddata 到内存时触发 |
| `indices.breaker.request.limit` | `60%` | 请求断路器。单次请求中聚合、排序等数据结构的内存使用 |
| `network.breaker.inflight_requests.limit` | `100%` | 传输中请求断路器。HTTP/Transport 层正在进行的请求占用的内存 |

> 百分比值相对于 JVM 堆内存（`-Xmx`）。

### 断路器触发时的错误

```json
{
  "error": {
    "type": "circuit_breaking_exception",
    "reason": "[parent] Data too large, data for [<operation>] would be [xxx/xxxmb], which is larger than the limit of [xxx/xxxmb]"
  }
}
```

**排查思路**：

1. 检查哪个断路器触发：`GET _nodes/stats/breaker`
2. 如果是 `fielddata` 断路器：优化查询，避免对 `text` 字段做聚合，改用 `keyword`
3. 如果是 `request` 断路器：减小聚合桶数（`max_buckets`）或分页聚合
4. 如果是 `total` 断路器：考虑增加堆内存或增加节点

---

## API 兼容

| 参数 | 默认值 | 属性 | 说明 |
|------|--------|------|------|
| `elasticsearch.api_compatibility` | `false` | 静态 | 是否启用 Elasticsearch API 兼容模式。启用后 Logstash、Filebeat 等 7.10.x OSS 客户端可直接连接 |
| `elasticsearch.api_compatibility_version` | `7.10.2` | 静态 | 兼容的 Elasticsearch 版本号。影响 `_cat`、`_cluster/health` 等 API 返回的版本信息 |

### 何时启用

- 从 Elasticsearch 迁移到 Easysearch 时，客户端代码暂时无法修改。
- 使用 Elastic Stack 生态工具（Logstash、Filebeat、Kibana OSS 等）直连 Easysearch。

```yaml
elasticsearch.api_compatibility: true
elasticsearch.api_compatibility_version: "7.10.2"
```

---

## 其他静态配置

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `script.painless.regex.enabled` | `true` | 是否允许在 Painless 脚本中使用正则表达式。正则可能有性能风险，高安全场景可关闭 |
| `bulk.max_actions` | 无限制 | 单次 `_bulk` 请求中允许的最大操作数 |

---

## 延伸阅读

- [集群配置]({{< relref "../configuration.md" >}}) — 断路器等动态设置的 API 操作
- [JVM 配置]({{< relref "./jvm.md" >}}) — 堆内存设置（影响断路器计算）
- [内存与缓存]({{< relref "./memory.md" >}}) — Fielddata 缓存设置

