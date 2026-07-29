---
title: "可变宽度直方图聚合（Variable Width Histogram）"
date: 0001-01-01
summary: "可变宽度直方图聚合 #  variable_width_histogram 聚合与标准 histogram 聚合类似，但它会自动调整每个桶的宽度，使数据点在各桶之间尽可能均匀分布。该聚合使用聚类算法，根据数据的实际分布动态确定最优的桶边界，而不是使用固定间隔。
这在数据分布不均匀时特别有用 —— 例如，大部分值集中在某个范围内，但也有少量离群值。使用固定间隔的 histogram 可能导致大量空桶或单个桶内文档过多，而 variable_width_histogram 能自适应地解决这些问题。
相关指南（先读这些） #    聚合基础  聚合场景实践  直方图聚合（固定宽度版本）  参数说明 #     参数 必需/可选 数据类型 描述     field 必填 String 要聚合的数值字段。必须为数值类型。也可使用 script 替代。   buckets 可选 Integer 期望的桶数量。实际返回的桶数量可能小于或等于此值。默认 10。必须大于 0。   shard_size 可选 Integer 每个分片上用于聚类的文档数量。值越大结果越精确，但内存消耗越大。默认为 buckets * 50。必须大于 1。   initial_buffer 可选 Integer 初始缓冲区大小，用于收集初始数据点以启动聚类算法。默认为 min(10 * shard_size, 50000)。必须大于 0 且不小于 buckets。   script 可选 Object 使用脚本动态生成聚合值。   missing 可选 Numeric 缺少字段值的文档所使用的替代值。    基本用法 #  以下示例将商品价格分成 5 个自适应宽度的桶："
---


# 可变宽度直方图聚合

`variable_width_histogram` 聚合与标准 `histogram` 聚合类似，但它会自动调整每个桶的宽度，使数据点在各桶之间尽可能均匀分布。该聚合使用聚类算法，根据数据的实际分布动态确定最优的桶边界，而不是使用固定间隔。

这在数据分布不均匀时特别有用 —— 例如，大部分值集中在某个范围内，但也有少量离群值。使用固定间隔的 `histogram` 可能导致大量空桶或单个桶内文档过多，而 `variable_width_histogram` 能自适应地解决这些问题。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})
- [直方图聚合]({{< relref "./histogram.md" >}})（固定宽度版本）

## 参数说明

| 参数             | 必需/可选 | 数据类型 | 描述                                                                                                                   |
| ---------------- | --------- | -------- | ---------------------------------------------------------------------------------------------------------------------- |
| `field`          | 必填      | String   | 要聚合的数值字段。必须为数值类型。也可使用 `script` 替代。                                                              |
| `buckets`        | 可选      | Integer  | 期望的桶数量。实际返回的桶数量可能小于或等于此值。默认 `10`。必须大于 0。                                                 |
| `shard_size`     | 可选      | Integer  | 每个分片上用于聚类的文档数量。值越大结果越精确，但内存消耗越大。默认为 `buckets * 50`。必须大于 1。                         |
| `initial_buffer` | 可选      | Integer  | 初始缓冲区大小，用于收集初始数据点以启动聚类算法。默认为 `min(10 * shard_size, 50000)`。必须大于 0 且不小于 `buckets`。  |
| `script`         | 可选      | Object   | 使用脚本动态生成聚合值。                                                                                                |
| `missing`        | 可选      | Numeric  | 缺少字段值的文档所使用的替代值。                                                                                        |

## 基本用法

以下示例将商品价格分成 5 个自适应宽度的桶：

```
GET products/_search
{
  "size": 0,
  "aggs": {
    "price_distribution": {
      "variable_width_histogram": {
        "field": "price",
        "buckets": 5
      }
    }
  }
}
```

返回内容中，每个桶包含 `min`、`max`、`key`（桶的中心值）和 `doc_count`：

```
{
  ...
  "aggregations": {
    "price_distribution": {
      "buckets": [
        {
          "min": 5.0,
          "key": 15.3,
          "max": 25.0,
          "doc_count": 120
        },
        {
          "min": 25.0,
          "key": 42.7,
          "max": 60.0,
          "doc_count": 95
        },
        {
          "min": 60.0,
          "key": 85.2,
          "max": 110.0,
          "doc_count": 88
        },
        {
          "min": 110.0,
          "key": 150.0,
          "max": 200.0,
          "doc_count": 45
        },
        {
          "min": 200.0,
          "key": 350.5,
          "max": 500.0,
          "doc_count": 12
        }
      ]
    }
  }
}
```

注意与固定间隔 `histogram` 的区别 —— 每个桶的宽度不同，但文档数量更加均匀。

## 调整精度

通过增加 `shard_size` 可以提高聚类精度（但会消耗更多内存）：

```
GET products/_search
{
  "size": 0,
  "aggs": {
    "price_distribution": {
      "variable_width_histogram": {
        "field": "price",
        "buckets": 10,
        "shard_size": 1000,
        "initial_buffer": 5000
      }
    }
  }
}
```

## histogram vs. variable_width_histogram

| 特性             | `histogram`                     | `variable_width_histogram`           |
| ---------------- | ------------------------------- | ------------------------------------ |
| 桶宽度           | 固定（由 `interval` 决定）       | 可变（由数据分布决定）                |
| 桶边界           | 均匀分布                        | 根据聚类算法自适应                    |
| 文档分布         | 可能极不均匀                    | 尽量均匀                              |
| 参数             | 需指定 `interval`               | 需指定 `buckets` 数量                 |
| 适用场景         | 数据分布均匀，需要固定区间       | 数据分布不均匀，需要自适应分桶         |

## 注意事项

1. `variable_width_histogram` 的结果是**近似的**，基于每个分片上的聚类算法。多分片环境下，结果可能因分片数据分布不同而有所差异。
2. `initial_buffer` 必须大于或等于 `buckets`，否则会报错。
3. `shard_size` 的 3/4 必须大于或等于 `buckets`，否则会报错。
4. 该聚合**不支持**子聚合（如嵌套 `avg` 或 `terms`）。
5. 仅支持数值类型字段。

