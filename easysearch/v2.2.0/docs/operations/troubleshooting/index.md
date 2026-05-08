---
title: "故障排查"
date: 0001-01-01
description: "问题诊断流程、日志分析、常见问题解决。"
summary: "故障排查 #  本文提供一条&quot;遇到问题时可以照着走&quot;的诊断路线，包含具体的诊断命令和解决方案。
 排查总体思路 #  遇到问题时，先快速判断三个问题：
┌─────────────────────────────────────────────────────────────────────────┐ │ 1. 影响范围：单个索引/租户？还是整个集群？ │ ├─────────────────────────────────────────────────────────────────────────┤ │ 2. 症状类型：性能下降？错误增多？节点异常？数据不符合预期？ │ ├─────────────────────────────────────────────────────────────────────────┤ │ 3. 时间维度：长期慢性问题？还是某个时间点突然恶化？ │ └─────────────────────────────────────────────────────────────────────────┘  问题 1：集群状态变红/黄 #  快速诊断 #  # 1. 检查集群状态 GET _cluster/health # 2. 查看未分配分片 GET _cat/shards?v&amp;h=index,shard,prirep,state,unassigned.reason&amp;s=state # 3. 分析未分配原因 GET _cluster/allocation/explain 常见原因与解决方案 #  原因 1：节点离线 #  诊断：
GET _cat/nodes?v 检查节点数是否符合预期，缺失的节点需要检查：
 进程是否存活：ps aux | grep easysearch 系统日志：journalctl -u easysearch Easysearch 日志：logs/{cluster_name}.log  解决方案："
---


# 故障排查

本文提供一条"遇到问题时可以照着走"的诊断路线，包含具体的诊断命令和解决方案。

---

## 排查总体思路

遇到问题时，先快速判断三个问题：

```
┌─────────────────────────────────────────────────────────────────────────┐
│  1. 影响范围：单个索引/租户？还是整个集群？                               │
├─────────────────────────────────────────────────────────────────────────┤
│  2. 症状类型：性能下降？错误增多？节点异常？数据不符合预期？              │
├─────────────────────────────────────────────────────────────────────────┤
│  3. 时间维度：长期慢性问题？还是某个时间点突然恶化？                      │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 问题 1：集群状态变红/黄

### 快速诊断

```json
# 1. 检查集群状态
GET _cluster/health

# 2. 查看未分配分片
GET _cat/shards?v&h=index,shard,prirep,state,unassigned.reason&s=state

# 3. 分析未分配原因
GET _cluster/allocation/explain
```

### 常见原因与解决方案

#### 原因 1：节点离线

**诊断**：
```json
GET _cat/nodes?v
```

检查节点数是否符合预期，缺失的节点需要检查：
- 进程是否存活：`ps aux | grep easysearch`
- 系统日志：`journalctl -u easysearch`
- Easysearch 日志：`logs/{cluster_name}.log`

**解决方案**：
1. 恢复故障节点
2. 如果节点无法恢复且有副本，集群会自动重新分配
3. 如果是主分片丢失且无副本，需要从快照恢复

#### 原因 2：磁盘空间不足

**诊断**：
```json
GET _cat/nodes?v&h=name,disk.used_percent,disk.avail
GET _cluster/allocation/explain
```

**典型响应**：
```json
{
  "deciders": [
    {
      "decider": "disk_threshold",
      "decision": "NO",
      "explanation": "the node is above the high watermark [90%]"
    }
  ]
}
```

**解决方案**：
```json
# 1. 查看磁盘使用情况
GET _cat/allocation?v

# 2. 清理旧索引（如果允许）
DELETE old-logs-2025*

# 3. 临时调整水位阈值（应急）
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.disk.watermark.low": "90%",
    "cluster.routing.allocation.disk.watermark.high": "95%",
    "cluster.routing.allocation.disk.watermark.flood_stage": "97%"
  }
}

# 4. 解除只读状态（如果被触发）
PUT _all/_settings
{
  "index.blocks.read_only_allow_delete": null
}
```

#### 原因 3：分片分配设置问题

**诊断**：
```json
# 检查索引设置
GET problematic-index/_settings?filter_path=*.settings.index.routing

# 检查集群设置
GET _cluster/settings?include_defaults=true&filter_path=*.cluster.routing
```

**常见问题**：
- 索引指定了不存在的节点属性
- 集群分配被禁用

**解决方案**：
```json
# 修复索引分配设置
PUT problematic-index/_settings
{
  "index.routing.allocation.include._name": null,
  "index.routing.allocation.require._name": null
}

# 重新启用集群分配
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "all"
  }
}
```

#### 原因 4：主分片丢失无法恢复

**诊断**：
```json
GET _cluster/allocation/explain
{
  "index": "problematic-index",
  "shard": 0,
  "primary": true
}
```

**最后手段**（会丢失数据）：
```json
# 分配空的主分片（仅在确认数据可以丢失时使用）
POST _cluster/reroute
{
  "commands": [
    {
      "allocate_empty_primary": {
        "index": "problematic-index",
        "shard": 0,
        "node": "node-1",
        "accept_data_loss": true
      }
    }
  ]
}
```

---

## 问题 2：搜索/写入延迟升高

### 快速诊断

```json
# 1. 检查节点资源
GET _cat/nodes?v&h=name,cpu,heap.percent,load_1m,disk.used_percent

# 2. 检查热点线程
GET _nodes/hot_threads

# 3. 检查任务积压
GET _cat/pending_tasks?v
GET _cat/thread_pool?v&h=node_name,name,active,queue,rejected
```

### 搜索延迟问题

#### 诊断：找出慢查询

```json
# 1. 配置慢查询日志
PUT /my-index/_settings
{
  "index.search.slowlog.threshold.query.warn": "5s",
  "index.search.slowlog.threshold.query.info": "2s"
}

# 2. 分析查询性能
GET /my-index/_search
{
  "profile": true,
  "query": {
    "match": { "content": "slow query" }
  }
}
```

#### 常见原因与解决方案

| 原因 | 诊断特征 | 解决方案 |
|------|----------|----------|
| 深度分页 | `from` 值很大 | 使用 `search_after` 替代 |
| 高基数聚合 | aggregation 耗时长 | 限制 `size`，使用近似聚合 |
| 通配符开头查询 | `wildcard` 以 `*` 开头 | 改用 `ngram` 或前缀匹配 |
| 大结果集 | `size` 值很大 | 减小 `size`，分批获取 |
| 分片过多 | 单查询命中数百分片 | 减少分片数，使用路由 |

#### 优化深度分页

```json
# 问题：深度分页性能差
GET /logs/_search
{
  "from": 10000,
  "size": 10,
  "query": { "match_all": {} }
}

# 解决：使用 search_after
GET /logs/_search
{
  "size": 10,
  "query": { "match_all": {} },
  "sort": [{ "@timestamp": "desc" }, { "_id": "asc" }],
  "search_after": ["2026-02-11T10:00:00.000Z", "doc-id-123"]
}
```

### 写入延迟问题

#### 诊断：检查写入队列

```json
# 1. 检查 bulk 队列
GET _cat/thread_pool/write?v&h=node_name,name,active,queue,rejected

# 2. 检查段合并压力
GET _cat/nodes?v&h=name,merges.current,merges.total

# 3. 检查 refresh/flush 状态
GET _cat/indices?v&h=index,refresh.time,flush.total
```

#### 常见原因与解决方案

| 原因 | 诊断特征 | 解决方案 |
|------|----------|----------|
| Bulk 批次过大 | 单次请求超过 100MB | 减小批次（5-15MB） |
| 并发过高 | queue 接近满，rejected > 0 | 降低写入并发 |
| Mapping 爆炸 | 字段数不断增长 | 限制 `total_fields.limit` |
| 频繁更新 | 大量 update/delete | 改为追加写入 + 版本控制 |
| 合并压力 | `merges.current` 持续高 | 调整 merge 策略 |

#### 优化写入性能

```json
# 1. 调整 refresh 间隔（写入期间）
PUT /my-index/_settings
{
  "index.refresh_interval": "30s"
}

# 2. 临时减少副本（批量导入期间）
PUT /my-index/_settings
{
  "index.number_of_replicas": 0
}

# 3. 调整 translog 策略
PUT /my-index/_settings
{
  "index.translog.durability": "async",
  "index.translog.sync_interval": "30s"
}
```

---

## 问题 3：节点频繁重启/离开集群

### 快速诊断

```bash
# 1. 检查系统日志
journalctl -u easysearch --since "1 hour ago"

# 2. 检查 Easysearch 日志
tail -f logs/easysearch.log | grep -E "ERROR|WARN|OOM"

# 3. 检查 GC 日志
cat logs/gc.log | grep "Full GC"
```

### 常见原因与解决方案

#### 原因 1：内存不足 / OOM

**诊断特征**：
```
java.lang.OutOfMemoryError: Java heap space
```

**解决方案**：
1. 检查堆内存设置：不超过可用内存的 50%，不超过 32GB
2. 检查是否有内存泄漏的查询（如超大聚合）
3. 增加节点或减少数据量

```bash
# 查看当前堆设置
grep -E "^-Xm[sx]" config/jvm.options

# 调整堆内存（重启生效）
# -Xms16g
# -Xmx16g
```

#### 原因 2：GC 压力过大

**诊断**：
```json
GET _nodes/stats/jvm?filter_path=nodes.*.jvm.gc
```

**解决方案**：
- 减小堆内存（如果接近 32GB）
- 检查是否有大量 Fielddata 加载
- 升级到 G1GC（推荐）

#### 原因 3：磁盘 I/O 瓶颈

**诊断**：
```bash
# 检查 I/O 等待
iostat -x 1

# 检查磁盘延迟
GET _nodes/stats/fs
```

**解决方案**：
- 升级到 SSD
- 检查是否有合并风暴
- 分散热点索引到多个节点

---

## 问题 4：查询结果不符合预期

### 搜不到应该存在的数据

**诊断**：
```json
# 1. 确认文档存在
GET /my-index/_doc/document-id

# 2. 检查 Mapping
GET /my-index/_mapping

# 3. 分析字段如何被分词
GET /my-index/_analyze
{
  "field": "content",
  "text": "搜索的关键词"
}

# 4. 检查查询如何解析
GET /my-index/_validate/query?explain
{
  "query": {
    "match": { "content": "关键词" }
  }
}
```

**常见原因**：

| 原因 | 诊断方法 | 解决方案 |
|------|----------|----------|
| 字段类型错误 | `_mapping` 显示 `keyword` 而非 `text` | 重建索引 |
| 分词器不匹配 | `_analyze` 结果与预期不符 | 调整 analyzer |
| refresh 延迟 | 刚写入的数据 | 等待或手动 refresh |
| 路由问题 | 使用了自定义路由 | 查询时指定路由 |

### 排序/相关性不合理

**诊断**：
```json
GET /my-index/_search
{
  "explain": true,
  "query": {
    "match": { "content": "关键词" }
  }
}
```

**查看每个文档的评分解释**，分析是否有意外的加权。

---

## 问题 5：Bulk 写入被拒绝

### 诊断

```json
# 1. 检查 rejected 计数
GET _cat/thread_pool/write?v

# 2. 检查队列设置
GET _cluster/settings?include_defaults=true&filter_path=*.thread_pool.write
```

**典型错误**：
```json
{
  "error": {
    "type": "es_rejected_execution_exception",
    "reason": "rejected execution of processing of [...]"
  }
}
```

### 解决方案

```json
# 1. 降低客户端并发
# 在客户端侧减少并发线程数

# 2. 减小 bulk 批次大小
# 每批 5-15MB，而不是 100MB

# 3. 增加队列大小（谨慎）
PUT _cluster/settings
{
  "transient": {
    "thread_pool.write.queue_size": 1000
  }
}

# 4. 客户端实现重试和指数退避
# 捕获 429/503 错误，等待后重试
```

---

## 诊断命令速查表

### 集群状态

```bash
# 整体健康
GET _cluster/health?pretty

# 节点列表
GET _cat/nodes?v&h=name,ip,heap.percent,ram.percent,cpu,load_1m,node.role

# 分片分布
GET _cat/shards?v&s=state

# 未分配原因
GET _cluster/allocation/explain

# 待处理任务
GET _cat/pending_tasks?v
```

### 性能诊断

```bash
# 热点线程
GET _nodes/hot_threads

# 线程池状态
GET _cat/thread_pool?v&h=node_name,name,active,queue,rejected

# 段信息
GET _cat/segments?v&h=index,shard,segment,size

# 索引统计
GET _cat/indices?v&s=store.size:desc
```

### 日志检查

```bash
# 查看错误日志
grep -E "ERROR|Exception" logs/easysearch.log | tail -100

# 查看慢查询日志
tail -f logs/easysearch_index_search_slowlog.log

# 查看 GC 日志
grep "Full GC" logs/gc.log
```

---

## 小结

| 问题类型 | 首要诊断 | 关键 API |
|----------|----------|----------|
| 集群变红 | 未分配分片原因 | `_cluster/allocation/explain` |
| 性能下降 | 资源使用 + 热点线程 | `_cat/nodes`、`_nodes/hot_threads` |
| 节点异常 | 系统日志 + GC 日志 | 操作系统层面 |
| 查询异常 | Mapping + Analyze | `_mapping`、`_analyze` |
| 写入拒绝 | 线程池队列 | `_cat/thread_pool` |

**排查原则**：
1. 先看影响面，再缩小范围
2. 先看监控趋势，再看实时状态
3. 先排除资源问题，再查业务问题
4. 保留现场（日志、配置），便于事后分析

**下一步阅读**：
- [监控]({{< relref "./monitoring" >}})：建立预警机制
- [容量规划]({{< relref "/docs/best-practices/index-design.md" >}})：避免资源瓶颈
- [配置]({{< relref "./configuration" >}})：优化集群配置

