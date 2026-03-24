---
title: "基数聚合（Cardinality）"
date: 0001-01-01
summary: "基数聚合 #  cardinality 聚合是一种单值指标聚合，用于计算字段的唯一值或不同值的数量。
基数计数为近似值。有关更多信息，请参阅下面的控制精度。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  cardinality 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需的 String 估计基数的字段。   precision_threshold 可选 Numeric 阈值，低于该阈值的计数预计接近准确值。有关更多信息，请参阅控制精度 。   execution_hint 可选 String 如何运行聚合，该参数会影响资源使用和聚合效率。有效值为 ordinals 和 direct 。   missing 可选 与 field 类型相同 用于存储字段缺失实例的 bucket。如果未提供，则忽略缺失值。    参考样例 #  以下示例请求查找数据中唯一产品 ID 的数量："
---


# 基数聚合

`cardinality` 聚合是一种单值指标聚合，用于计算字段的唯一值或不同值的数量。

基数计数为近似值。有关更多信息，请参阅下面的控制精度。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`cardinality` 聚合采用以下参数。

| 参数                  | 必需/可选 | 数据类型            | 描述                                                                             |
| --------------------- | --------- | ------------------- | -------------------------------------------------------------------------------- |
| `field`               | 必需的    | String              | 估计基数的字段。                                                                 |
| `precision_threshold` | 可选      | Numeric             | 阈值，低于该阈值的计数预计接近准确值。有关更多信息，请参阅控制精度 。            |
| `execution_hint`      | 可选      | String              | 如何运行聚合，该参数会影响资源使用和聚合效率。有效值为 `ordinals` 和 `direct` 。 |
| `missing`             | 可选      | 与 `field` 类型相同 | 用于存储字段缺失实例的 bucket。如果未提供，则忽略缺失值。                        |

## 参考样例

以下示例请求查找数据中唯一产品 ID 的数量：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "unique_products": {
      "cardinality": {
        "field": "products.product_id"
      }
    }
  }
}
```

## 返回内容

如以下内容所示，聚合返回 `unique_products` 变量中的基数计数：

```
{
  "took": 176,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "unique_products": {
      "value": 7033
    }
  }
}
```

## 控制精度

准确的基数计算需要将所有值加载到哈希集中并返回其大小。这种方法扩展性较差；它可能需要大量内存并导致高延迟。

您可以使用 `precision_threshold` 设置来控制内存和准确度之间的权衡。此参数设置一个阈值，低于该阈值的计数预计会接近准确度。高于此值的计数可能会降低准确度。

`precision_threshold` 的默认值为 3000，支持的最大值为 40000。

基数聚合使用 [HyperLogLog++ 算法](https://static.googleusercontent.com/media/research.google.com/fr//pubs/archive/40671.pdf) 。基数计数通常在精度阈值以下非常准确，并且在大多数情况下，即使阈值低至 100，其与真实计数的误差也在 6% 以内。

### 预计算哈希

对于高基数字符串字段，存储索引字段的哈希值并计算哈希值的基数可以节省计算和内存资源。请谨慎使用此方法；它仅适用于长字符串和/或高基数的集合。数值字段和内存消耗较低的字符串集合最好直接处理。

### 示例：控制精度

将精度阈值设置为 `10000` 唯一值：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "unique_products": {
      "cardinality": {
        "field": "products.product_id",
        "precision_threshold": 10000
      }
    }
  }
}
```

返回内容与使用默认阈值的结果类似，但返回值略有不同。可以调整 `precision_threshold` 参数，以查看它如何影响基数估计。


## 缺省值处理

可以为聚合字段的缺失内容分配一个值。有关详细信息，请参阅缺省聚合。

替换基数聚合中的缺失值会将替换值添加到唯一值列表中，从而将实际基数增加 1。

