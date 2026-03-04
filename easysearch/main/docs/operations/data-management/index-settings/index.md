---
title: "索引设置（Index Settings）"
date: 0001-01-01
description: "影响性能和存储的常用索引级别设置。"
summary: "索引设置（Index Settings） #  索引级别的设置直接影响写入、查询性能以及资源利用率。本页按功能分类列出关键设置及其默认值，帮助你在建索引或调优时有一个清单。
查看与修改索引设置 #  // 查看索引的所有设置 GET /my-index/_settings // 查看特定设置 GET /my-index/_settings/index.refresh_interval // 动态修改设置（仅限 Dynamic 类型的设置） PUT /my-index/_settings { &#34;index.refresh_interval&#34;: &#34;30s&#34; }  Static vs Dynamic：Static 设置只能在索引创建时指定或在关闭索引后修改；Dynamic 设置可以在运行时通过 _settings API 随时修改。
 分片与副本 #     设置 默认值 类型 说明     index.number_of_shards 1 Static 主分片数量。创建后不可更改（上限由 es.index.max_number_of_shards 控制，默认 1024）   index.number_of_replicas 1 Dynamic 每个主分片的副本数量。最小值 0   index.number_of_routing_shards 等于主分片数 Static 用于 _split 操作的路由分片数，必须 ≥ number_of_shards   index."
---


# 索引设置（Index Settings）

索引级别的设置直接影响写入、查询性能以及资源利用率。本页按功能分类列出关键设置及其默认值，帮助你在建索引或调优时有一个清单。

## 查看与修改索引设置

```json
// 查看索引的所有设置
GET /my-index/_settings

// 查看特定设置
GET /my-index/_settings/index.refresh_interval

// 动态修改设置（仅限 Dynamic 类型的设置）
PUT /my-index/_settings
{
  "index.refresh_interval": "30s"
}
```

> **Static vs Dynamic**：Static 设置只能在索引创建时指定或在关闭索引后修改；Dynamic 设置可以在运行时通过 `_settings` API 随时修改。

## 分片与副本

| 设置 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `index.number_of_shards` | `1` | Static | 主分片数量。创建后不可更改（上限由 `es.index.max_number_of_shards` 控制，默认 1024） |
| `index.number_of_replicas` | `1` | Dynamic | 每个主分片的副本数量。最小值 0 |
| `index.number_of_routing_shards` | 等于主分片数 | Static | 用于 `_split` 操作的路由分片数，必须 ≥ `number_of_shards` |
| `index.auto_expand_replicas` | `false` | Dynamic | 根据集群节点数自动扩展副本，例如 `"0-5"` 或 `"0-all"` |
| `index.routing_partition_size` | `1` | Dynamic | 自定义路由时可以路由到的分片子集大小 |

### 设计建议

- **主分片数** 创建后无法修改（取模路由会变），需要扩容时通过 **新索引 + [重建索引](./reindex.md) + [别名切换](./aliases.md)** 实现
- 对中小数据量索引，避免设置过多主分片；推荐按时间/业务拆索引
- 副本越多读吞吐越高，但写入与存储成本也越大；写多读少的索引可适当降低副本数

创建索引时指定分片与副本：

```json
PUT /my-index
{
  "settings": {
    "index.number_of_shards": 3,
    "index.number_of_replicas": 2
  }
}
```

## 刷新与近实时

| 设置 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `index.refresh_interval` | `1s` | Dynamic | 刷新间隔。设为 `"-1"` 可关闭自动刷新 |

刷新控制"近实时"行为——新写入的文档需要经过一次 refresh 才能被 `_search` 检索到（详见 [写入与存储机制]({{< relref "/docs/fundamentals/write-and-storage" >}})）。

**常见调优场景**：

- 高写入吞吐（如批量导入）：临时设为 `"-1"` 关闭自动刷新，导入完成后手动刷新
- 对实时性要求不高的索引：设为 `"30s"` 或 `"60s"` 以减少刷新开销

```json
// 批量导入时关闭自动刷新
PUT /my-index/_settings
{ "index.refresh_interval": "-1" }

// 导入完成后恢复并手动刷新
PUT /my-index/_settings
{ "index.refresh_interval": "1s" }

POST /my-index/_refresh
```

## 事务日志（Translog）

事务日志用于保证数据持久性——在 Lucene 提交（flush）之前，所有写入都记录在 translog 中，以便节点崩溃后恢复。

| 设置 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `index.translog.durability` | `REQUEST` | Dynamic | `REQUEST`：每次写入后 fsync（安全但较慢）；`ASYNC`：异步 fsync（更快但有丢失风险） |
| `index.translog.sync_interval` | `5s` | Dynamic | `ASYNC` 模式下的 fsync 间隔 |
| `index.translog.flush_threshold_size` | `512mb` | Dynamic | 当 translog 达到此大小时触发 flush（Lucene 提交） |

**调优建议**：

- 默认 `REQUEST` 模式最安全，每次写入返回成功即表示数据已持久化
- 如果可以接受极端情况（节点突然断电）下丢失最近几秒的写入，可改为 `ASYNC` 以提升写入性能
- `flush_threshold_size` 越大，flush 频率越低，但恢复时间也越长

## 段合并（Merge）

写入产生的小段（segment）会被后台合并为更大的段，以维持查询效率。合并策略影响 IO 与 CPU 开销。

### 合并策略（Merge Policy）

| 设置 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `index.merge.policy.floor_segment` | `2mb` | Dynamic | 小于此值的段会被优先合并 |
| `index.merge.policy.max_merge_at_once` | `10` | Dynamic | 一次合并最多合并的段数（最小 2） |
| `index.merge.policy.max_merged_segment` | `5gb` | Dynamic | 合并后单个段的最大大小；超过此值的段不再参与合并 |
| `index.merge.policy.expunge_deletes_allowed` | `10.0` | Dynamic | 强制合并时允许的已删除文档占比 |
| `index.merge.policy.segments_per_tier` | `10.0` | Dynamic | 每层允许的段数（最小 2.0） |
| `index.merge.policy.deletes_pct_allowed` | `33.0` | Dynamic | 段中已删除文档的最大占比（20.0–50.0），超过则强制合并 |

### 合并调度器（Merge Scheduler）

| 设置 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `index.merge.scheduler.max_thread_count` | `max(1, min(4, CPU/2))` | Dynamic | 并发合并线程数，基于节点 CPU 核数自动计算 |
| `index.merge.scheduler.max_merge_count` | `max_thread_count + 5` | Dynamic | 最大待合并数量 |
| `index.merge.scheduler.auto_throttle` | `true` | Dynamic | 是否自动调节合并 IO 速率 |

> 一般情况下，默认策略已经足够。应通过监控观察是否存在长期高合并压力或段数量异常；只有在确有问题时才针对性调整。

## 编解码器与压缩（Codec）

| 设置 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `index.codec` | `"default"` | Static | 数据压缩编解码器。可选值：`"default"`（LZ4）、`"best_compression"`（DEFLATE，压缩比更高但速度稍慢）、`"lucene_default"` |
| `index.codec.compression_level` | （默认未设置） | Static | 压缩级别，仅部分 codec 支持 |

选择建议：

- **写入密集型 / 对查询延迟敏感**：使用默认的 `"default"`（LZ4）
- **存储成本敏感 / 冷数据归档**：使用 `"best_compression"`（DEFLATE）

更多关于索引压缩的讨论，参见 [索引压缩](./index-compression.md)。

## 查询与结果限制

| 设置 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `index.max_result_window` | `10000` | Dynamic | `from + size` 的最大值（深分页上限） |
| `index.max_inner_result_window` | `100` | Dynamic | 内部命中（inner_hits / top_hits 聚合）的 `from + size` 上限 |
| `index.max_rescore_window` | `10000` | Dynamic | rescore 请求的 `window_size` 上限，默认与 `max_result_window` 一致 |
| `index.max_docvalue_fields_search` | `100` | Dynamic | 查询中允许的 `docvalue_fields` 最大数量 |
| `index.max_script_fields` | `32` | Dynamic | 查询中允许的 `script_fields` 最大数量 |
| `index.max_terms_count` | `65536` | Dynamic | Terms 查询中允许的最大 term 数量 |
| `index.max_regex_length` | `1000` | Dynamic | 正则表达式查询的最大长度 |

## 其他重要设置

| 设置 | 默认值 | 类型 | 说明 |
|------|--------|------|------|
| `index.blocks.read_only` | `false` | Dynamic | 设为 `true` 则索引变为只读 |
| `index.blocks.read_only_allow_delete` | `false` | Dynamic | 只读但允许删除（磁盘空间不足时系统自动设置） |
| `index.blocks.read` | `false` | Dynamic | 禁止读操作 |
| `index.blocks.write` | `false` | Dynamic | 禁止写操作 |
| `index.blocks.metadata` | `false` | Dynamic | 禁止元数据修改 |
| `index.hidden` | `false` | Dynamic | 隐藏索引，不会被通配符匹配到 |
| `index.default_pipeline` | `"_none"` | Dynamic | 默认的[摄取管道]({{< relref "/docs/features/ingest-pipelines/_index.md" >}})，所有写入自动经过该管道 |
| `index.final_pipeline` | `"_none"` | Dynamic | 最终管道，在 `default_pipeline` 之后执行，不可被请求级别覆盖 |
| `index.search.idle.after` | `30s` | Dynamic | 搜索空闲超时时间，超过后跳过后台刷新以节省资源 |
| `index.highlight.max_analyzed_offset` | `1000000` | Dynamic | 高亮分析的最大字符偏移量 |

## 模板与默认设置

为了保持索引配置的一致性，通常使用[索引模板](./index-templates.md)来：

- 为某类索引（按名称模式匹配）统一设置分片数、副本数、刷新策略等
- 统一分析器、动态映射策略

在多租户或多环境（开发/测试/生产）场景下，建议通过模板与自动化来管理索引设置，避免手工创建索引时遗漏关键参数。

## 小结

- `index.number_of_shards` 和 `index.number_of_replicas` 是最基础的设置，直接影响数据分布与可用性
- `index.refresh_interval` 控制近实时可见性，是写入性能调优的首要旋钮
- `index.translog.durability` 在性能与持久性之间做权衡
- 段合并设置一般保持默认，除非监控显示合并压力异常
- `index.codec` 在存储成本与查询速度之间选择
- 用[索引模板](./index-templates.md)统一管理设置，避免逐个索引手动配置

下一步可以继续阅读：

- [别名（Aliases）](./aliases.md)
- [容量规划]({{< relref "/docs/best-practices/index-design" >}})
- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics" >}})



