---
title: "加权平均聚合（Weighted Avg）"
date: 0001-01-01
summary: "加权平均聚合 #  weighted_avg 聚合计算跨文档数值的加权平均值。当您想计算平均值，但希望某些数据点的权重大于其他数据点时，此功能非常有用。
相关指南（先读这些） #    聚合基础  聚合场景实践  加权平均值使用公式 $\frac{\sum_{i=1}^n value_i \cdot weight_i}{\sum_{i=1}^n weight_i}$ 计算。
参数说明 #  weighted_avg 聚合采用以下参数。
   参数 必需/可选 描述     value 必需 定义如何获取要计算平均值的数值。需要 field 或 script 。   weight 必需 定义如何获取每个值的权重。需要 field 或 script 。   format 可选 DecimalFormat 格式字符串。返回聚合的 value_as_string 属性中的格式化输出。   value_type 可选 使用脚本或未映射字段时的值的类型提示。    可以在 value 或 weight 内指定以下参数。"
---


# 加权平均聚合

`weighted_avg` 聚合计算跨文档数值的加权平均值。当您想计算平均值，但希望某些数据点的权重大于其他数据点时，此功能非常有用。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

加权平均值使用公式 $\frac{\sum_{i=1}^n value_i \cdot weight_i}{\sum_{i=1}^n weight_i}$ 计算。

## 参数说明

`weighted_avg` 聚合采用以下参数。

| 参数         | 必需/可选 | 描述                                                                      |
| ------------ | --------- | ------------------------------------------------------------------------- |
| `value`      | 必需      | 定义如何获取要计算平均值的数值。需要 `field` 或 `script` 。               |
| `weight`     | 必需      | 定义如何获取每个值的权重。需要 `field` 或 `script` 。                     |
| `format`     | 可选      | DecimalFormat 格式字符串。返回聚合的 value_as_string 属性中的格式化输出。 |
| `value_type` | 可选      | 使用脚本或未映射字段时的值的类型提示。                                    |

可以在 `value` 或 `weight` 内指定以下参数。

| 参数      | 必需/可选 | 描述                                         |
| --------- | --------- | -------------------------------------------- |
| `field`   | 可选      | 用于值或权重的文档字段。                     |
| `missing` | 可选      | 字段缺失时使用的默认值或权重。请参阅缺失值。 |

## 参考样例

首先，创建索引并索引一些数据。请注意，产品 C 缺少 `rating` 和 `num_reviews` 字段：

```
POST _bulk
{ "index": { "_index": "products" } }
{ "name": "Product A", "rating": 4.5, "num_reviews": 100 }
{ "index": { "_index": "products" } }
{ "name": "Product B", "rating": 3.8, "num_reviews": 50 }
{ "index": { "_index": "products" } }
{ "name": "Product C"}

```

以下请求使用 `weighted_avg` 聚合来计算加权平均产品评分。在此上下文中，每个产品的评分都由其 `num_reviews` 加权。这意味着，评论较多的产品对最终平均值的影响将大于评论较少的产品：

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "weighted_rating": {
      "weighted_avg": {
        "value": {
          "field": "rating"
        },
        "weight": {
          "field": "num_reviews"
        },
        "format": "#.##"
      }
    }
  }
}
```

## 返回内容

响应包含 `weighted_rating` ，计算结果为 `weighted_avg = (4.5 * 100 + 3.8 * 50) / (100 + 50) = 4.27` 。仅考虑同时包含 `rating` 和 `num_reviews` 值的文档 1 和 2：

```
{
  "took": 18,
  "timed_out": false,
  "terminated_early": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "weighted_rating": {
      "value": 4.266666650772095,
      "value_as_string": "4.27"
    }
  }
}
```

## 缺省值

使用 `missing` 参数，您可以为缺少 `value` 字段或 `weight` 字段的文档指定默认值，而不是将它们从计算中排除。

例如，您可以为没有评级的产品分配 3.0 的“平均”评级，并将 `num_reviews` 设置为 1，以赋予它们较小的非零权重：

```
GET /products/_search
{
  "size": 0,
  "aggs": {
    "weighted_rating": {
      "weighted_avg": {
        "value": {
          "field": "rating",
          "missing": 3.0
        },
        "weight": {
          "field": "num_reviews",
          "missing": 1
        },
        "format": "#.##"
      }
    }
  }
}

```

新的加权平均值计算为 `weighted_avg = (4.5 * 100 + 3.8 * 50 + 3.0 * 1) / (100 + 50 + 1) = 4.26` ：

```
{
  "took": 27,
  "timed_out": false,
  "terminated_early": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "weighted_rating": {
      "value": 4.258278129906055,
      "value_as_string": "4.26"
    }
  }
}
```

