---
title: "索引排序"
date: 0001-01-01
summary: "索引排序配置指南 #  索引排序（Index Sorting）允许在索引阶段对文档按指定字段排序存储。这可以显著加速按排序字段的查询和聚合操作，但会增加索引写入开销。
 工作原理 #  默认情况下，Lucene 按文档写入顺序存储。启用索引排序后，每个段（Segment）内的文档将按排序字段有序存储：
默认存储： doc1 → doc2 → doc3 → doc4 → doc5 索引排序后： doc3 → doc1 → doc5 → doc2 → doc4 (按 timestamp 降序) 优势：
 按排序字段查询时可提前终止（Early Termination），大幅减少扫描量 提高按排序字段做聚合的效率 改善压缩率（相邻文档字段值更接近）  代价：
 索引写入速度降低 ~40%–50%（段合并时需要排序） 索引大小可能增加 排序字段只能在索引创建时指定，无法修改   配置参数 #  index.sort.field #     项目 说明     参数 index.sort.field   默认值 无（不排序）   属性 静态（仅在索引创建时设置）   说明 排序字段列表。支持多字段排序。字段类型必须是 boolean、numeric、date 或 keyword    index."
---


# 索引排序配置指南

索引排序（Index Sorting）允许在索引阶段对文档按指定字段排序存储。这可以显著加速按排序字段的查询和聚合操作，但会增加索引写入开销。

---

## 工作原理

默认情况下，Lucene 按文档写入顺序存储。启用索引排序后，每个段（Segment）内的文档将按排序字段有序存储：

```
默认存储：   doc1 → doc2 → doc3 → doc4 → doc5
索引排序后： doc3 → doc1 → doc5 → doc2 → doc4  (按 timestamp 降序)
```

**优势：**
- 按排序字段查询时可提前终止（Early Termination），大幅减少扫描量
- 提高按排序字段做聚合的效率
- 改善压缩率（相邻文档字段值更接近）

**代价：**
- 索引写入速度降低 ~40%–50%（段合并时需要排序）
- 索引大小可能增加
- 排序字段只能在索引创建时指定，无法修改

---

## 配置参数

### index.sort.field

| 项目 | 说明 |
|------|------|
| **参数** | `index.sort.field` |
| **默认值** | 无（不排序） |
| **属性** | 静态（仅在索引创建时设置） |
| **说明** | 排序字段列表。支持多字段排序。字段类型必须是 `boolean`、`numeric`、`date` 或 `keyword` |

### index.sort.order

| 项目 | 说明 |
|------|------|
| **参数** | `index.sort.order` |
| **默认值** | `asc` |
| **可选值** | `asc`、`desc` |
| **属性** | 静态 |
| **说明** | 每个排序字段的排序方向。元素个数必须与 `index.sort.field` 一致 |

### index.sort.mode

| 项目 | 说明 |
|------|------|
| **参数** | `index.sort.mode` |
| **默认值** | `min`（asc 时）/ `max`（desc 时） |
| **可选值** | `min`、`max` |
| **属性** | 静态 |
| **说明** | 多值字段的取值模式。`min` 取最小值排序，`max` 取最大值排序 |

### index.sort.missing

| 项目 | 说明 |
|------|------|
| **参数** | `index.sort.missing` |
| **默认值** | `_last`（asc 时）/ `_first`（desc 时） |
| **可选值** | `_last`、`_first` |
| **属性** | 静态 |
| **说明** | 缺失值的排列位置。`_first` 排在最前，`_last` 排在最后 |

---

## 使用示例

### 按时间戳降序排序

最常见的场景——日志/时序数据按时间降序存储，加速"最新数据"查询：

```bash
PUT /logs-2024
{
  "settings": {
    "index": {
      "sort.field": "timestamp",
      "sort.order": "desc",
      "number_of_shards": 3,
      "number_of_replicas": 1
    }
  },
  "mappings": {
    "properties": {
      "timestamp": {
        "type": "date"
      },
      "level": {
        "type": "keyword"
      },
      "message": {
        "type": "text"
      }
    }
  }
}
```

### 多字段排序

按租户 ID + 时间戳排序，适用于多租户场景：

```bash
PUT /multi-tenant-index
{
  "settings": {
    "index": {
      "sort.field": ["tenant_id", "timestamp"],
      "sort.order": ["asc", "desc"],
      "sort.missing": ["_last", "_first"]
    }
  },
  "mappings": {
    "properties": {
      "tenant_id": {
        "type": "keyword"
      },
      "timestamp": {
        "type": "date"
      }
    }
  }
}
```

---

## 提前终止（Early Termination）

索引排序的核心收益是查询提前终止。当查询的 `sort` 顺序与索引排序一致时，Easysearch 可以在收集到足够文档后提前停止扫描：

```bash
# 索引按 timestamp desc 排序
# 查询也按 timestamp desc 排序 + size=10
# → 每个段只需读取前 10 个文档
GET /logs-2024/_search
{
  "size": 10,
  "sort": [
    { "timestamp": "desc" }
  ],
  "query": {
    "match_all": {}
  }
}
```

**效果对比（示例）：**

| 场景 | 无索引排序 | 有索引排序 |
|------|:----------:|:----------:|
| Top 10 最新日志 | 扫描全部文档 | 每段仅读 10 条 |
| 按时间范围 + Top N | 全段扫描 | 快速跳过无关数据 |
| 聚合（按排序字段分桶） | 全量计算 | 可利用有序性优化 |

> ⚠️ **提前终止仅在查询 sort 与索引 sort 完全匹配时生效**。如果查询按其他字段排序，索引排序不会带来查询收益。

---

## 适用场景

### ✅ 推荐使用

| 场景 | 排序配置 | 理由 |
|------|---------|------|
| 日志/时序数据 | `timestamp desc` | 大多数查询是"最新 N 条" |
| 多租户 | `tenant_id asc, timestamp desc` | 按租户隔离查询 |
| 电商订单 | `created_at desc` | 最新订单优先 |
| 传感器数据 | `device_id asc, timestamp desc` | 按设备查最新数据 |

### ❌ 不推荐使用

| 场景 | 理由 |
|------|------|
| 高写入吞吐 | 索引排序增加 40–50% 写入开销 |
| 全文搜索为主 | 查询不按固定字段排序，无法提前终止 |
| 排序字段频繁变化 | 索引排序一旦设定无法修改 |
| `text` 类型字段排序 | 不支持 |

---

## 性能影响

### 写入性能

索引排序的主要开销在段合并时的排序操作：

| 指标 | 无排序 | 有排序 | 影响 |
|------|:------:|:------:|:----:|
| 索引吞吐 | 基准 | -40%~-50% | 显著下降 |
| 段合并时间 | 基准 | +50%~+100% | 显著增加 |
| 索引大小 | 基准 | +5%~-10% | 取决于数据（有序数据可能压缩更好） |

### 查询性能

匹配排序方向时收益显著：

| 查询类型 | 无排序 | 有排序 | 提升 |
|---------|:------:|:------:|:----:|
| Top N（排序匹配） | 全量扫描 | 提前终止 | **10x–100x** |
| 范围查询 + 排序 | 全量扫描 | 减少扫描 | 2x–10x |
| 不匹配的排序 | 基准 | 无收益 | 无 |
| 聚合（排序字段） | 基准 | 有序优化 | 2x–5x |

---

## 注意事项

1. **索引排序不可修改**：排序字段和方向在索引创建时确定，无法通过 `_settings` API 修改
2. **Reindex 保留排序**：对排序索引执行 reindex，目标索引需要重新配置排序
3. **字段类型限制**：仅支持 `boolean`、`numeric`、`date`、`keyword` 类型
4. **与 `_source` 无关**：索引排序影响 Lucene 段内的存储顺序，`_source` 中的字段顺序不变
5. **副本同步**：主分片和副本分片的段排序一致

---

## 延伸阅读

- [路径配置]({{< relref "../config/node-settings/path.md" >}}) - 数据存储路径配置
- [高级配置参数]({{< relref "./advanced-settings.md" >}}) - 完整参数参考

