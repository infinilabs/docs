---
title: "时间序列索引优化"
date: 0001-01-01
summary: "时间序列索引优化 #   最低版本：1.12.1
 概述 #  在处理时序数据（如日志、监控指标、事件流）时，数据通常具有明显的时间先后顺序。Easysearch 底层的 Lucene Segment 合并是保证搜索性能和资源效率的关键操作。 然而，默认的合并策略（TieredMergePolicy）主要基于 Segment 的大小和删除文档比例来决定合并哪些 Segment，它并不感知数据的时间属性。
对于时序场景，默认策略可能导致：
 冷热数据混合合并：较旧的（冷）数据 Segment 可能与较新的（热）数据 Segment 合并，导致不必要的 I/O 和 CPU 开销。 查询性能下降：跨时间范围的大 Segment 可能降低按时间范围过滤的查询效率。  为此，Easysearch 引入了基于时间范围的合并策略（TimeRangeMergePolicy），专为时序索引优化 Segment 合并行为。
核心原理 #  TimeRangeMergePolicy 在选择要合并的 Segment 时，除了考虑大小、删除比例等因素外，优先考虑 Segment 所覆盖的时间范围：
 时间优先：倾向于合并时间上相邻的 Segment，保持数据的&quot;时间局部性&quot;。 保留时间分区：避免将时间跨度很大的 Segment 合并在一起。 优先合并新数据：新写入的数据变化更频繁，优先合并较新的 Segment 有助于更快回收空间和优化查询性能。  工作流程 #   每个 Segment 在创建时记录 min_timestamp 和 max_timestamp。 合并候选按最新时间倒序排列（同一时间的按大小排列）。 候选合并组的评分以时间跨度为主要因子 — 时间跨度越小评分越优。 单次合并限制在 max_merge_at_once 个 Segment 以内，合并后大小不超过 max_merged_segment。   如何启用 #  设置索引的 index."
---


# 时间序列索引优化

> **最低版本**：1.12.1

## 概述

在处理时序数据（如日志、监控指标、事件流）时，数据通常具有明显的时间先后顺序。Easysearch 底层的 Lucene Segment 合并是保证搜索性能和资源效率的关键操作。
然而，默认的合并策略（TieredMergePolicy）主要基于 Segment 的大小和删除文档比例来决定合并哪些 Segment，它并不感知数据的时间属性。

对于时序场景，默认策略可能导致：

- **冷热数据混合合并**：较旧的（冷）数据 Segment 可能与较新的（热）数据 Segment 合并，导致不必要的 I/O 和 CPU 开销。
- **查询性能下降**：跨时间范围的大 Segment 可能降低按时间范围过滤的查询效率。

为此，Easysearch 引入了基于时间范围的合并策略（**TimeRangeMergePolicy**），专为时序索引优化 Segment 合并行为。

## 核心原理

TimeRangeMergePolicy 在选择要合并的 Segment 时，除了考虑大小、删除比例等因素外，**优先考虑 Segment 所覆盖的时间范围**：

- **时间优先**：倾向于合并时间上相邻的 Segment，保持数据的"时间局部性"。
- **保留时间分区**：避免将时间跨度很大的 Segment 合并在一起。
- **优先合并新数据**：新写入的数据变化更频繁，优先合并较新的 Segment 有助于更快回收空间和优化查询性能。

### 工作流程

1. 每个 Segment 在创建时记录 `min_timestamp` 和 `max_timestamp`。
2. 合并候选按**最新时间倒序**排列（同一时间的按大小排列）。
3. 候选合并组的评分以**时间跨度**为主要因子 — 时间跨度越小评分越优。
4. 单次合并限制在 `max_merge_at_once` 个 Segment 以内，合并后大小不超过 `max_merged_segment`。

---

## 如何启用

设置索引的 `index.merge.policy.time_range_field` 为时间字段名即可启用。

### 对已有索引启用

```json
PUT /my-timeseries-index/_settings
{
  "index": {
    "merge.policy.time_range_field": "@timestamp"
  }
}
```

### 通过索引模板默认启用

```json
PUT /_index_template/timeseries_template
{
  "index_patterns": ["logs-*", "metrics-*"],
  "template": {
    "settings": {
      "index.merge.policy.time_range_field": "@timestamp"
    }
  }
}
```

> **注意**：`time_range_field` 指定的字段必须是 `date` 类型，且在索引 Mapping 中已定义。

---

## 配置参考

### 核心设置

| 设置                                           | 描述                                                | 类型     | 默认值 | 动态 |
| ---------------------------------------------- | --------------------------------------------------- | -------- | ------ | ---- |
| `index.merge.policy.time_range_field`          | 时间字段名。设置后自动启用 TimeRangeMergePolicy。      | `string` | `""`   | 否   |

### 合并行为设置

以下设置适用于 TimeRangeMergePolicy 和默认的 TieredMergePolicy：

| 设置                                           | 描述                                                     | 类型       | 默认值 | 范围        | 动态 |
| ---------------------------------------------- | -------------------------------------------------------- | ---------- | ------ | ----------- | ---- |
| `index.merge.policy.max_merge_at_once`         | 单次合并的最大 Segment 数量。                              | `int`      | `10`   | ≥ 2         | 是   |
| `index.merge.policy.max_merged_segment`        | 合并产生的 Segment 最大大小。超过此大小的 Segment 不参与合并。| `ByteSize` | `5gb`  | ≥ 0         | 是   |
| `index.merge.policy.segments_per_tier`         | 每层允许的 Segment 数量。值越小合并越频繁、Segment 越少。    | `double`   | `10.0` | ≥ 2.0       | 是   |
| `index.merge.policy.floor_segment`             | 小于此大小的 Segment 会被"取整"到此大小参与合并评分。        | `ByteSize` | `2mb`  | > 0         | 是   |
| `index.merge.policy.expunge_deletes_allowed`   | 删除文档百分比超过此阈值的 Segment 才会被强制合并清理。       | `double`   | `10.0` | ≥ 0.0       | 是   |
| `index.merge.policy.deletes_pct_allowed`       | 索引中允许的最大删除文档百分比。                            | `double`   | `33.0` | 20.0 ~ 50.0 | 是   |

### 高级设置

| 设置                                           | 描述                                                     | 类型    | 默认值 | 状态     |
| ---------------------------------------------- | -------------------------------------------------------- | ------- | ------ | -------- |
| `index.merge.policy.max_merge_at_once_explicit` | force merge 时的最大合并 Segment 数量。                    | `int`   | `30`   | 已弃用   |
| `index.merge.policy.reclaim_deletes_weight`     | 旧版删除回收权重。                                        | `double`| `2.0`  | 已弃用   |
| `index.compound_format`                         | 复合文件系统比率。超过此比例的 Segment 不使用复合文件格式。  | `double`| `0.1`  | —        |

---

## 优势

- **降低合并开销**：显著减少冷热数据混合合并，降低不必要的 I/O 和 CPU 消耗。
- **提高资源效率**：更智能的合并可能降低存储空间占用（更快回收已删除空间）。
- **优化查询性能**：保持 Segment 的时间局部性，提升时间范围过滤查询的速度。
- **对时序数据友好**：特别适合日志、指标等严格按时间增长的数据模式。

## 注意事项

- **时间字段依赖**：必须正确指定一个能代表数据时间的 `date` 类型字段。如果字段不存在或数据时间分布混乱，策略效果可能不佳。
- **适用场景**：最适用于具有明显时间序列特征的数据。对于非时序数据，默认的 TieredMergePolicy 可能更合适。
- **不可动态切换**：`time_range_field` 设置不支持动态修改，需要在索引创建时或关闭索引后设置。
- **Force Merge**：执行 force merge 时，TimeRangeMergePolicy 会移除最大 Segment 大小限制，允许合并到目标段数。

---

## 完整示例

### 创建优化的时序索引

```json
PUT /metrics-2024
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index.merge.policy.time_range_field": "@timestamp",
    "index.merge.policy.max_merged_segment": "10gb",
    "index.merge.policy.segments_per_tier": 8
  },
  "mappings": {
    "properties": {
      "@timestamp": {
        "type": "date"
      },
      "host": {
        "type": "keyword"
      },
      "cpu_usage": {
        "type": "float"
      }
    }
  }
}
```

### 结合生命周期管理使用

在索引模板中同时启用时间范围合并策略和 ILM 策略：

```json
PUT /_index_template/metrics_template
{
  "index_patterns": ["metrics-*"],
  "template": {
    "settings": {
      "number_of_shards": 3,
      "index.merge.policy.time_range_field": "@timestamp",
      "index.lifecycle.name": "metrics_lifecycle",
      "index.lifecycle.rollover_alias": "metrics"
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "host": { "type": "keyword" },
        "cpu_usage": { "type": "float" },
        "mem_usage": { "type": "float" }
      }
    }
  }
}
```

