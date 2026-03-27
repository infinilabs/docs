---
title: "高级配置参数"
date: 0001-01-01
summary: "高级配置参数 #  本页面详细列举 Easysearch 的所有高级配置参数，包括集群、节点、索引等多层级配置。
 概述 #  Easysearch 配置涉及多个层级：
 节点级：影响单个节点的行为（node., path., 等） 集群级：影响整个集群的行为（cluster., discovery., 等） 索引级：影响单个索引的行为（index.*, 等） 传输层：网络通信相关（transport., http., 等）   集群协调配置 #  集群选举、节点发现和主节点选择的相关配置。
选举配置 #     参数 默认值 动态 说明     cluster.election.initial_timeout 100ms ✗ 初始选举超时   cluster.election.back_off_time 100ms ✗ 选举退避时间   cluster.election.max_timeout 10s ✗ 最大选举超时   cluster.election.duration 30s ✗ 选举持续时间    主节点检查配置 #     参数 默认值 动态 说明     cluster."
---


# 高级配置参数

本页面详细列举 Easysearch 的所有高级配置参数，包括集群、节点、索引等多层级配置。

---

## 概述

Easysearch 配置涉及多个层级：

- **节点级**：影响单个节点的行为（node.*, path.*, 等）
- **集群级**：影响整个集群的行为（cluster.*, discovery.*, 等）
- **索引级**：影响单个索引的行为（index.*, 等）
- **传输层**：网络通信相关（transport.*, http.*, 等）

---

## 集群协调配置

集群选举、节点发现和主节点选择的相关配置。

### 选举配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `cluster.election.initial_timeout` | 100ms | ✗ | 初始选举超时 |
| `cluster.election.back_off_time` | 100ms | ✗ | 选举退避时间 |
| `cluster.election.max_timeout` | 10s | ✗ | 最大选举超时 |
| `cluster.election.duration` | 30s | ✗ | 选举持续时间 |

### 主节点检查配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `cluster.leader_check.interval` | 1s | ✗ | 主节点检查间隔 |
| `cluster.leader_check.timeout` | 10s | ✗ | 主节点检查超时 |
| `cluster.leader_check.retry_count` | 3 | ✗ | 主节点检查重试次数 |
| `cluster.follower_check.interval` | 1s | ✗ | 追随节点检查间隔 |
| `cluster.follower_check.timeout` | 10s | ✗ | 追随节点检查超时 |
| `cluster.follower_check.retry_count` | 3 | ✗ | 追随节点检查重试次数 |

### 集群形成配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `cluster.initial_master_nodes` | - | ✗ | 初始主节点列表（引导集群时必需） |
| `cluster.bootstrap_unconfigured_timeout` | 15s | ✗ | 集群初始化超时 |
| `cluster.publish_info.timeout` | 30s | ✗ | 发布信息超时 |
| `cluster.publish.timeout` | 30s | ✗ | 发布配置超时 |
| `cluster.auto_shrink_voting_configuration` | true | ✓ | 自动收缩投票配置 |

### 故障检测配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `cluster.follower_lag_timeout` | 60s | ✗ | 节点滞后超时 |
| `cluster.formation_warning_timeout` | 10s | ✗ | 集群形成警告超时 |
| `discovery.cluster_formation_warning_timeout` | 10s | ✗ | 同上（别名） |

---

## 路由和分片分配

### 磁盘阈值配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `cluster.routing.allocation.disk.threshold_enabled` | true | ✓ | 启用磁盘阈值检查 |
| `cluster.routing.allocation.disk.watermark.low` | 85% | ✓ | 低水位线 |
| `cluster.routing.allocation.disk.watermark.high` | 90% | ✓ | 高水位线 |
| `cluster.routing.allocation.disk.watermark.flood_stage` | 95% | ✓ | 泛滥水位线 |
| `cluster.routing.allocation.disk.include_relocations` | true | ✓ | 磁盘检查是否包含重定位 |
| `cluster.routing.allocation.disk.reroute_interval` | 60s | ✓ | 磁盘检查重新路由间隔 |
| `cluster.routing.allocation.disk.enable_for_single_data_node` | false | ✗ | 单节点模式下启用磁盘检查 |

> 磁盘阈值也支持绝对字节值：`1TB`, `500GB` 等

### 分片分配配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `cluster.routing.allocation.enable` | all | ✓ | 分片分配允许的操作 |
| `cluster.routing.allocation.same_shard.host` | false | ✓ | 禁止同一主分片和副本在同一主机 |
| `cluster.routing.allocation.cluster_concurrent_recoveries` | -1 | ✓ | 并发恢复数（-1=无限制） |
| `cluster.routing.allocation.max_retries` | 5 | ✗ | 分片分配最大重试次数 |
| `cluster.routing.allocation.awareness.attributes` | - | ✓ | 机架感知属性列表 |
| `cluster.routing.rebalance.enable` | all | ✓ | 集群重新平衡允许的操作 |
| `index.routing.allocation.enable` | all | ✓ | 索引分片分配（仅限该索引） |
| `index.routing.allocation.initial_recovery.*` | - | ✗ | 初始恢复时的分配限制 |
| `index.total_shards_per_node` | -1 | ✓ | 单节点最大分片数（-1=无限制） |
| `cluster.total_shards_per_node` | -1 | ✓ | 集群级别：单节点最大分片数 |
| `cluster.routing.allocation.same_shard.host` | false | ✓ | 相同分片的副本不能在同一主机 |

### 重新平衡配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `cluster.routing.rebalance.enable` | all | ✓ | 允许重新平衡 |
| `cluster.routing.allocation.allow_rebalance` | indices_primaries_active | ✓ | 何时允许重新平衡 |

---

## 恢复配置

恢复失败分片数据的相关配置。

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `indices.recovery.max_bytes_per_sec` | 40mb | ✓ | 恢复带宽限制 |
| `indices.recovery.max_concurrent_operations` | 1 | ✓ | 并发操作数 |
| `indices.recovery.max_concurrent_file_chunks` | 2 | ✓ | 并发文件块数 |
| `indices.recovery.retry_delay_state_sync` | 5s | ✓ | 状态同步重试延迟 |
| `indices.recovery.retry_delay_network` | 5s | ✓ | 网络重试延迟 |
| `indices.recovery.internal_action_timeout` | 15m | ✓ | 内部恢复操作超时 |
| `indices.recovery.internal_action_long_timeout` | 30m | ✓ | 长期恢复操作超时 |
| `indices.recovery.activity_timeout` | 15m | ✓ | 恢复活动超时 |
| `indices.recovery.max_concurrent_recoveries` | 2 | ✓ | 并发恢复数（已弃用，使用allocation配置） |
| `index.delayed_node_left_timeout` | 1m | ✓ | 节点离线后延迟重新分配时间 |

---

## 索引存储配置

### 存储类型

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.store.type` | fs | ✗ | 存储类型（fs=文件系统） |
| `index.store.fs.fs_lock` | native | ✗ | 文件系统锁类型 |
| `node.store.allow_mmap` | true | ✗ | 允许使用 mmap |
| `index.store.hybrid.chunk_size` | 128MB | ✗ | 混合存储块大小 |
| `index.optimize_auto_generated_id` | true | ✗ | 优化自动生成的文档 ID |

### 事务日志配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.translog.durability` | request | ✓ | 事务日志持久化策略 |
| `index.translog.sync_interval` | 5s | ✓ | 事务日志同步间隔 |
| `index.translog.flush_threshold_size` | 512mb | ✓ | 事务日志刷新阈值 |
| `index.translog.retention.size` | -1 | ✓ | 事务日志保留大小 |
| `index.translog.retention.age` | -1 | ✓ | 事务日志保留时间 |

### Soft Delete（软删除）

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.soft_deletes.enabled` | true | ✗ | 启用软删除 |
| `index.soft_deletes.retention_operations` | -1 | ✓ | 软删除保留操作数 |
| `index.soft_deletes.retention.lease.period` | 12h | ✓ | 软删除保留租期 |

---

## 缓存配置

### Query Cache（查询缓存）

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `indices.queries.cache.size` | 10% | ✓ | 查询缓存大小 |
| `indices.queries.cache.count` | 10000 | ✓ | 缓存条目数限制 |
| `indices.queries.cache.all_segments` | false | ✓ | 是否缓存所有 segment 过滤器 |
| `index.queries.cache.enabled` | true | ✓ | 该索引是否启用查询缓存 |

### Request Cache（请求缓存）

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `indices.requests.cache.size` | 1% | ✓ | 请求缓存大小 |
| `indices.requests.cache.expire` | -1 | ✓ | 缓存过期时间 |
| `index.requests.cache.enable` | true | ✓ | 该索引是否启用请求缓存 |

---

## 断路器配置

断路器防止节点 OOM（内存溢出）。

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `indices.breaker.total.limit` | 95% | ✓ | 总断路器限制 |
| `indices.breaker.fielddata.limit` | 40% | ✓ | Field Data 断路器 |
| `indices.breaker.fielddata.overhead` | 1.03 | ✓ | Field Data 开销系数 |
| `indices.breaker.request.limit` | 60% | ✓ | 请求断路器 |
| `indices.breaker.request.overhead` | 1.0 | ✓ | 请求开销系数 |
| `indices.breaker.accounting.limit` | 100% | ✓ | 会计断路器 |
| `indices.breaker.accounting.overhead` | 1.0 | ✓ | 会计开销系数 |
| `network.breaker.inflight_requests.limit` | 100% | ✓ | 传输请求断路器 |
| `network.breaker.inflight_requests.overhead` | 2.0 | ✓ | 传输请求开销系数 |
| `indices.breaker.type` | hierarchy | ✗ | 断路器类型 |
| `indices.breaker.total.use_real_memory` | true | ✓ | 使用真实内存计算 |

---

## 搜索相关配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.query.default_field` | - | ✓ | 默认查询字段 |
| `indices.query.bool.max_clause_count` | 1024 | ✓ | Bool 查询最大子句数 |
| `search.max_open_pit_context` | 300 | ✓ | 最大 PIT 上下文数 |
| `search.max_keep_alive` | 24h | ✓ | 搜索/滚动最大保活时间 |
| `point_in_time.max_keep_alive` | 24h | ✓ | PIT 最大保活时间 |
| `index.search.slowlog.threshold.query.warn` | -1 | ✓ | 搜索慢日志警告阈值 |
| `index.search.slowlog.threshold.query.info` | -1 | ✓ | 搜索慢日志信息阈值 |
| `index.search.slowlog.threshold.fetch.warn` | -1 | ✓ | Fetch 慢日志警告阈值 |
| `index.search.slowlog.threshold.fetch.info` | -1 | ✓ | Fetch 慢日志信息阈值 |
| `index.search.throttled` | false | ✓ | 限制该索引的搜索速度 |

---

## 索引配置

### 基础设置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.number_of_shards` | 1 | ✗ | 主分片数 |
| `index.number_of_replicas` | 1 | ✓ | 副本数 |
| `index.codec` | best_compression | ✗ | 压缩编码器 |
| `index.force_memory_term_dictionary` | false | ✗ | 强制 term dictionary 使用内存 |

### 索引排序

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.sort.field` | - | ✗ | 索引排序字段 |
| `index.sort.order` | asc | ✗ | 索引排序顺序 |
| `index.sort.missing` | _last | ✗ | 缺失值处理 |

### 刷新和合并

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.refresh_interval` | 1s | ✓ | 刷新间隔 |
| `index.max_result_window` | 10000 | ✓ | 最大结果窗口（from+size） |
| `index.max_inner_result_window` | 100000 | ✓ | 最大内部结果窗口 |
| `index.max_rescore_window` | 50000 | ✓ | 最大重新评分窗口 |
| `index.max_docvalue_fields_search` | 200 | ✓ | 最大 docvalue 字段数 |
| `index.max_script_fields` | 32 | ✓ | 最大脚本字段数 |
| `index.default_pipeline` | - | ✓ | 默认数据管道 |

### 合并策略

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.merge.policy.type` | tiered | ✗ | 合并策略 |
| `index.merge.policy.segments_per_tier` | 10 | ✓ | 分层合并层级大小 |
| `index.merge.policy.max_merge_at_once` | 30 | ✓ | 最大合并数 |
| `index.merge.policy.max_merge_at_once_explicit` | 30 | ✓ | 明确请求时的最大合并数 |
| `index.merge.policy.floor_segment` | 2mb | ✓ | 最小合并段大小 |
| `index.merge.policy.max_merged_segment` | 5gb | ✓ | 最大合并段大小 |
| `index.merge.policy.deletes_pct_allowed_in_commits` | 10 | ✓ | 允许的删除百分比 |
| `index.merge.scheduler.max_thread_count` | - | ✗ | 合并线程数 |
| `index.merge.scheduler.auto_throttle` | true | ✓ | 自动限流 |

---

## 映射和分析配置

### 映射相关

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.mapping.total_fields.limit` | 2000 | ✓ | 最大字段数 |
| `index.mapping.depth.limit` | 20 | ✓ | 最大嵌套深度 |
| `index.mapping.nested_fields.limit` | 50 | ✓ | 最大嵌套字段数 |
| `index.mapping.nested_objects.limit` | 10000 | ✓ | 最大嵌套对象数 |
| `index.verified_before_close` | false | ✓ | 关闭前验证索引 |

### 分析器

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `index.analysis.hunspell.dictionary.lazy` | false | ✗ | Hunspell 字典延迟加载 |
| `index.analysis.hunspell.dictionary.ignore_case` | false | ✗ | Hunspell 忽略大小写 |

---

## 数据管道（Ingest）配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `ingest.geoip.cache_size` | 1000 | ✗ | GeoIP 缓存大小 |

---

## 传输层配置

### TCP 连接

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `transport.tcp.connect_timeout` | 30s | ✗ | TCP 连接超时 |
| `transport.profiles.default.connect_timeout` | 30s | ✗ | 默认传输配置文件超时 |
| `transport.ping_schedule` | -1 | ✗ | 心跳 ping 间隔 |

### 远程集群配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `cluster.remote.connect` | true | ✗ | 允许连接远程集群（已弃用） |
| `cluster.remote.node.attr` | - | ✗ | 远程集群节点属性 |
| `search.remote.fetch_size` | 100 | ✓ | 远程搜索的 fetch 大小 |

---

## HTTP 层配置

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `http.host` | - | ✗ | HTTP 绑定地址 |
| `http.port` | 9200 | ✗ | HTTP 端口 |
| `http.max_content_length` | 100mb | ✗ | 最大请求体大小 |
| `http.max_header_size` | 16kb | ✗ | 最大 HTTP 头大小 |
| `http.max_initial_line_length` | 4kb | ✗ | 最大初始行长度 |
| `http.compression` | true | ✗ | 启用 gzip 压缩 |
| `http.compression_level` | 6 | ✗ | 压缩级别 |
| `http.publish_host` | - | ✗ | HTTP 发布地址 |
| `http.bind_host` | - | ✗ | HTTP 绑定地址 |
| `http.publish_port` | - | ✗ | HTTP 发布端口 |
| `http.reset_cookies` | false | ✗ | 重置 Cookie |

---

## 监控配置

### JVM 监控

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `monitor.jvm.gc.enabled` | true | ✗ | 启用 JVM GC 监控 |

### 文件系统监控

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `monitor.fs.health.enabled` | true | ✓ | 启用文件系统健康检查 |
| `monitor.fs.always_refresh` | false | ✗ | 总是刷新文件系统信息 |

---

## 网关（Gateway）配置

网关负责节点启动时恢复分片数据。

| 参数 | 默认值 | 动态 | 说明 |
|------|-------|------|------|
| `gateway.recover_after_nodes` | - | ✗ | 启动后等待节点数达到此值后才开始恢复 |
| `gateway.recover_after_time` | 5m | ✗ | 启动后等待此时间后开始恢复 |
| `gateway.expected_nodes` | -1 | ✗ | 预期节点数（已弃用） |
| `gateway.slow_write_logging_threshold` | 10s | ✗ | 缓慢写入日志阈值 |

---

## 搜索和线程池

### 线程池配置（按名称）

| 线程池 | 线程类型 | 队列类型 | 说明 |
|-------|--------|--------|------|
| search | scaling | resizable | 搜索线程池 |
| index | fixed | resizable | 索引线程池 |
| bulk | fixed | resizable | 批量操作 |
| get | fixed | resizable | 单文档获取 |
| analyze | fixed | resizable | 分析器 |
| write | fixed | resizable | 写入操作 |
| snapshot | scaling | resizable | 快照操作 |
| force_merge | fixed | fixed | 强制合并 |
| fetch_shard_store | fixed | fixed | 获取分片存储 |
| management | scaling | resizable | 管理操作 |

### 线程池参数

| 参数格式 | 说明 |
|---------|------|
| `thread_pool.{name}.type` | 线程池类型（fixed/scaling） |
| `thread_pool.{name}.size` | 线程池大小 |
| `thread_pool.{name}.queue_size` | 队列大小 |
| `thread_pool.{name}.keep_alive` | 线程保活时间 |

---

## 安全配置

参考配置文件说明中的安全配置部分

---

## 系统级别配置

### 进程和性能

| 参数 | 默认值 | 说明 |
|------|-------|------|
| `node.max_local_storage_nodes` | 1 | 单机最大节点数 |
| `node.pidfile` | - | PID 文件路径 |
| `node.store.allow_mmap` | true | 允许 mmap |

### 引导参数

| 参数 | 默认值 | 说明 |
|------|-------|------|
| `bootstrap.memory_lock` | false | 锁定 JVM 内存 |
| `bootstrap.system_call_filter.enabled` | true | 启用系统调用过滤 |
| `bootstrap.system_call_filter.all` | false | 过滤所有系统调用 |

---

## 常见优化建议

### 高吞吐量集群

```yaml
# 禁用请求缓存（如果大量不同查询）
indices.requests.cache.size: 0%

# 增大查询缓存
indices.queries.cache.size: 15%

# 调整刷新间隔
index.refresh_interval: 30s

# 增加合并线程
index.merge.scheduler.max_thread_count: 4
```

### 内存受限环境

```yaml
# 降低缓存大小
indices.queries.cache.size: 5%
indices.requests.cache.size: 0.5%

# 启用断路器保护
indices.breaker.total.limit: 70%

# 减少刷新间隔（更多内存使用）
index.refresh_interval: 5s

# 降低合并并发
index.merge.policy.max_merge_at_once: 10
```

### 搜索优化

```yaml
# 启用请求缓存
index.requests.cache.enable: true

# 调整搜索超时
search.default_search_timeout: 30s

# 禁用 PIT 自动清理（如需要）
search.max_open_pit_context: 1000
```

---

## 配置更新方法

### 从文件配置（需重启）

编辑 `config/easysearch.yml` 或 `config/easysearch.properties`，修改后重启节点。

### 动态更新（推荐）

使用集群设置 API 动态更新标记为"✓"的参数：

```json
PUT /_cluster/settings
{
  "transient": {
    "indices.queries.cache.size": "15%",
    "index.refresh_interval": "30s"
  }
}
```

**持久化更新**（集群重启后仍生效）：

```json
PUT /_cluster/settings
{
  "persistent": {
    "indices.queries.cache.size": "15%"
  }
}
```

---

## 延伸阅读

- [基础配置]({{< relref "../config/configuration.md" >}})
- [集群配置]({{< relref "../config/configuration.md" >}})
- [JVM 配置]({{< relref "../config/node-settings/jvm.md" >}})
- [内存管理]({{< relref "../config/node-settings/memory.md" >}})
- [日志配置]({{< relref "../config/node-settings/log4j2.md" >}})

