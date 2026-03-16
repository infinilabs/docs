---
title: "集群配置"
date: 0001-01-01
summary: "集群配置 #  本文介绍通过集群设置 API 管理 Easysearch 集群级别配置的方式，包括动态设置的查看、修改、重置，以及可通过 API 调整的全部动态配置项参考。
 节点级别的静态配置（需要写在 easysearch.yml 中、重启后生效的配置）请参见 节点配置。
  集群设置 API #  大多数集群级调优参数都可以通过集群设置 API 在运行时更改，无需重启节点。
查看设置 #  # 查看所有设置（包含默认值） GET _cluster/settings?include_defaults=true # 仅查看用户自定义设置 GET _cluster/settings 设置优先级 #  集群设置 API 中存在三类设置：持久（Persistent）、临时（Transient）和默认（Default）。
优先级从高到低：
 Transient 设置 — 临时，集群重启后丢失 Persistent 设置 — 持久，跨重启保留 配置文件 easysearch.yml 默认值   推荐做法：节点相关配置放 easysearch.yml，集群范围的调优通过 API 以 persistent 方式修改，避免每个节点单独改配置文件。
 修改设置 #  // 持久化修改（推荐用于集群调优） PUT /_cluster/settings { &#34;persistent&#34;: { &#34;action."
---


# 集群配置

本文介绍通过集群设置 API 管理 Easysearch 集群级别配置的方式，包括动态设置的查看、修改、重置，以及可通过 API 调整的全部动态配置项参考。

> 节点级别的静态配置（需要写在 `easysearch.yml` 中、重启后生效的配置）请参见 [节点配置]({{< relref "./node-settings/" >}})。

---

## 集群设置 API

大多数集群级调优参数都可以通过集群设置 API 在运行时更改，无需重启节点。

### 查看设置

```bash
# 查看所有设置（包含默认值）
GET _cluster/settings?include_defaults=true

# 仅查看用户自定义设置
GET _cluster/settings
```

### 设置优先级

集群设置 API 中存在三类设置：持久（Persistent）、临时（Transient）和默认（Default）。

优先级从高到低：

1. **Transient** 设置 — 临时，集群重启后丢失
2. **Persistent** 设置 — 持久，跨重启保留
3. 配置文件 `easysearch.yml`
4. 默认值

> 推荐做法：节点相关配置放 `easysearch.yml`，集群范围的调优通过 API 以 `persistent` 方式修改，避免每个节点单独改配置文件。

### 修改设置

```json
// 持久化修改（推荐用于集群调优）
PUT /_cluster/settings
{
  "persistent": {
    "action.auto_create_index": false
  }
}

// 临时修改（适合调试，重启后失效）
PUT /_cluster/settings
{
  "transient": {
    "logger.org.easysearch.discovery": "DEBUG"
  }
}
```

### 重置设置

将配置值设为 `null` 即可重置为默认值：

```json
PUT /_cluster/settings
{
  "persistent": {
    "action.auto_create_index": null
  }
}
```

---

## 动态配置项参考

以下配置项均可通过 `PUT /_cluster/settings` 在线修改，无需重启节点。

### 索引行为

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `action.auto_create_index` | `true` | 是否允许自动创建索引。可设为 `false` 禁止，或指定允许的模式如 `+logs-*,-*` |
| `action.destructive_requires_name` | `false` | 是否要求删除索引时必须指定具体名称，禁止通配符和 `_all`。**生产环境建议设为 `true`** |

### 索引默认值

> 以下为集群级别的索引默认值，作用于**新创建的索引**。部分参数也可以通过 `PUT /<index>/_settings` 在索引级别动态覆盖（标注"可动态修改"的）。

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `index.number_of_shards` | `1` | 新建索引的默认主分片数。**注意：索引创建后主分片数不可修改** |
| `index.number_of_replicas` | `1` | 新建索引的默认副本数（可动态修改） |
| `index.refresh_interval` | `1s` | 索引刷新间隔（文档写入后多久可被搜索到）。设为 `-1` 可禁用自动刷新 |
| `index.translog.durability` | `request` | Translog 刷写策略。`request`（每次请求同步刷写，最安全）或 `async`（异步刷写，更快但有丢失风险） |
| `index.translog.flush_threshold_size` | `512mb` | Translog 累积到此大小触发 flush 到 Lucene |
| `index.max_result_window` | `10000` | 搜索结果窗口上限（`from + size` 的最大值）。过大会占用大量堆内存 |

### 分片分配

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `cluster.routing.allocation.enable` | `all` | 分片分配开关。可选：`all`、`primaries`（仅主分片）、`new_primaries`（仅新主分片）、`none`（禁止）。**滚动重启时设为 `primaries` 或 `none`** |
| `cluster.routing.allocation.node_concurrent_recoveries` | `2` | 单节点并发恢复分片数 |
| `cluster.routing.allocation.node_concurrent_incoming_recoveries` | `2` | 单节点并发接收恢复分片数 |
| `cluster.routing.allocation.node_concurrent_outgoing_recoveries` | `2` | 单节点并发发送恢复分片数 |
| `cluster.max_shards_per_node` | `1000` | 单节点最大分片数（所有索引分片总和） |

### 分片分配感知

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `cluster.routing.allocation.awareness.attributes` | — | 分片分配感知属性。配合 `node.attr.*` 使用，确保主分片和副本分布在不同区域。例：`zone` |
| `cluster.routing.allocation.awareness.force.zone.values` | — | 强制感知属性值列表。确保即使部分区域不可用也不会将所有副本分配到同一区域。例：`["zone-a", "zone-b"]` |

### 磁盘水位线

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `cluster.routing.allocation.disk.threshold_enabled` | `true` | 是否启用基于磁盘使用量的分片分配 |
| `cluster.routing.allocation.disk.watermark.low` | `85%` | 低水位线：磁盘使用超过此阈值后，不再向该节点分配新分片（已有分片保留不动） |
| `cluster.routing.allocation.disk.watermark.high` | `90%` | 高水位线：超过后不再分配新分片，并开始将已有分片迁移到其他节点 |
| `cluster.routing.allocation.disk.watermark.flood_stage` | `95%` | 洪水位线：超过后相关索引变为只读 |
| `cluster.routing.allocation.disk.reroute_interval` | `60s` | 磁盘水位线检测间隔（多久检查一次并进行重新分配） |
| `cluster.routing.allocation.disk.include_relocations` | `true` | 是否在磁盘水位线计算中包含正在进行中的副本搬运（已弃用） |

示例：调整磁盘水位线

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.disk.watermark.low": "80%",
    "cluster.routing.allocation.disk.watermark.high": "88%",
    "cluster.routing.allocation.disk.watermark.flood_stage": "93%"
  }
}
```

### 分片分配过滤

用于节点退役前迁走所有分片：

| 参数 | 说明 |
|------|------|
| `cluster.routing.allocation.exclude._name` | 按节点名称排除 |
| `cluster.routing.allocation.exclude._ip` | 按 IP 地址排除 |
| `cluster.routing.allocation.exclude._host` | 按主机名排除 |
| `cluster.routing.allocation.include._name` | 仅允许分配到指定节点 |
| `cluster.routing.allocation.require._name` | 要求必须分配到指定节点 |

示例：下线节点前迁移分片

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.exclude._name": "node-3"
  }
}
```

### 分片恢复限流

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `indices.recovery.max_bytes_per_sec` | `40mb` | 分片恢复时每个节点的最大传输速率。SSD 可适当调高 |
| `indices.store.throttle.max_bytes_per_sec` | `10mb` | 段合并限流。SSD 环境可设为 `200mb` 以上 |

示例：提高恢复速率（SSD 环境）

```json
PUT /_cluster/settings
{
  "persistent": {
    "indices.recovery.max_bytes_per_sec": "200mb"
  }
}
```

### 分片延迟分配

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `index.unassigned.node_left.delayed_timeout` | `1m` | 节点离开后，推迟分片重新分配的时间。避免因短暂网络抖动导致不必要的数据搬运。可通过索引设置 API 修改 |

### 内存与缓存

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `indices.fielddata.cache.size` | 无限制 | 字段数据缓存上限（百分比或绝对值），超过阈值时触发回收 |

### 断路器

断路器防止操作导致 OutOfMemoryError：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `indices.breaker.total.limit` | `70%` | 总断路器上限（所有断路器的聚合限制） |
| `indices.breaker.fielddata.limit` | `40%` | 字段数据断路器（加载 fielddata 到内存时触发） |
| `indices.breaker.request.limit` | `60%` | 请求断路器（单次请求中聚合等数据结构） |
| `network.breaker.inflight_requests.limit` | `100%` | 传输中请求断路器（HTTP/Transport 层正在进行的请求） |

示例：调整断路器

```json
PUT /_cluster/settings
{
  "persistent": {
    "indices.breaker.total.limit": "75%",
    "indices.breaker.fielddata.limit": "45%"
  }
}
```

### 跨集群搜索

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `cluster.remote.<alias>.seeds` | — | 远程集群的种子节点地址列表 |
| `cluster.remote.<alias>.transport.compress` | `false` | 是否压缩跨集群通信 |
| `cluster.remote.<alias>.skip_unavailable` | `false` | 远程集群不可用时是否跳过（而非报错） |

示例：添加远程集群

```json
PUT /_cluster/settings
{
  "persistent": {
    "cluster.remote.cluster_b.seeds": ["192.168.2.10:9300", "192.168.2.11:9300"],
    "cluster.remote.cluster_b.transport.compress": true,
    "cluster.remote.cluster_b.skip_unavailable": true
  }
}
```

### 日志级别

可在不重启节点的情况下动态调整任意 Logger 的级别：

```json
// 开启 DEBUG 日志
PUT /_cluster/settings
{
  "transient": {
    "logger.org.easysearch.discovery": "DEBUG"
  }
}

// 恢复默认级别
PUT /_cluster/settings
{
  "transient": {
    "logger.org.easysearch.discovery": null
  }
}
```

### 慢日志阈值

慢日志可以按索引设置阈值，帮助发现性能问题：

```json
PUT /my-index/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s",
  "index.search.slowlog.threshold.fetch.warn": "1s",
  "index.indexing.slowlog.threshold.index.warn": "10s",
  "index.indexing.slowlog.threshold.index.info": "5s"
}
```

---

## 延伸阅读

- [节点配置]({{< relref "./node-settings/" >}}) — `easysearch.yml` 静态配置项详细参考
- [系统调优]({{< relref "./settings.md" >}}) — 操作系统内核参数
- [硬件配置]({{< relref "./hardware.md" >}}) — CPU / 内存 / 磁盘 / 网络选型
- [安全配置]({{< relref "/docs/operations/security/" >}}) — 用户、角色、权限管理

