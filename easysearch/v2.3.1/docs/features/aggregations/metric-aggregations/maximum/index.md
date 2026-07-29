---
title: "最大值聚合（Max）"
date: 0001-01-01
summary: "最大值聚合 #  max 聚合是一个单值指标聚合，返回字段的最大值。
相关指南（先读这些） #    聚合基础  聚合场景实践   max 聚合使用 double （双精度）表示来比较数值字段。对于包含 long 或 unsigned_long 超过 2^53 的整数值的字段，结果应被视为近似值，因为 double 的尾数中的有效位数是 53。
 参数说明 #  max 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 计算最大值的字段名称。   missing 可选 Numeric 分配给字段缺失实例的值。如果未提供，则包含缺失值的文档将从聚合中排除。    参考样例 #  以下示例请求在数据中查找最昂贵的商品——即 base_unit_price 值最大的商品：
GET sample_data_ecommerce/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;max_base_unit_price&#34;: { &#34;max&#34;: { &#34;field&#34;: &#34;products."
---


# 最大值聚合

`max` 聚合是一个单值指标聚合，返回字段的最大值。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

> max 聚合使用 double （双精度）表示来比较数值字段。对于包含 long 或 unsigned_long 超过 2^53 的整数值的字段，结果应被视为近似值，因为 double 的尾数中的有效位数是 53。

## 参数说明

`max` 聚合采用以下参数。

| 参数      | 必需/可选 | 数据类型 | 描述                                                                   |
| --------- | --------- | -------- | ---------------------------------------------------------------------- |
| `field`   | 必需      | String   | 计算最大值的字段名称。                                                 |
| `missing` | 可选      | Numeric  | 分配给字段缺失实例的值。如果未提供，则包含缺失值的文档将从聚合中排除。 |

## 参考样例

以下示例请求在数据中查找最昂贵的商品——即 `base_unit_price` 值最大的商品：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "max_base_unit_price": {
      "max": {
        "field": "products.base_unit_price"
      }
    }
  }
}
```

## 返回内容

如以下示例返回内容所示，聚合返回 `products.base_unit_price` 的最大值：

```
{
  "took": 24,
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
    "max_base_unit_price": {
      "value": 540
    }
  }
}

```

您可以使用聚合名称（ `max_base_unit_price` ）作为键从响应中检索聚合。

## 缺省值

您可以为聚合字段的缺失实例分配一个值。有关更多信息，请参阅缺失聚合。
缺省值通常被 `max` 忽略。如果您使用 `missing` 分配一个大于任何现有值的值， `max` 将返回此替换值作为最大值。

