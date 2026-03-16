---
title: "矩阵统计聚合（Matrix Stats）"
date: 0001-01-01
summary: "矩阵统计聚合 #  matrix_stats 聚合是一个多值指标聚合，以矩阵形式为两个或多个字段生成协方差统计。
 注意：matrix_stats 聚合不支持脚本。
 相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  matrix_stats 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     field 必需 String 用于计算矩阵统计的一组字段。   missing 可选 Object 用于替代缺失值的值。默认情况下，会忽略缺失值。参见缺失值。   mode 可选 String 用作多值或数组字段样本的值。允许的值是 avg 、 min 、 max 、 sum 和 median 。默认是 avg 。    参考样例 #  以下示例返回数据中 taxful_total_price 和 products."
---


# 矩阵统计聚合

`matrix_stats` 聚合是一个多值指标聚合，以矩阵形式为两个或多个字段生成协方差统计。

> **注意**：`matrix_stats` 聚合不支持脚本。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`matrix_stats` 聚合采用以下参数。
| 参数 | 必需/可选 | 数据类型 | 描述 |
| ---------------- | --------- | -------- | ------------------------------------------------------ |
| `field` | 必需 | String | 用于计算矩阵统计的一组字段。 |
| `missing` | 可选 | Object | 用于替代缺失值的值。默认情况下，会忽略缺失值。参见缺失值。 |
| `mode` | 可选 | String | 用作多值或数组字段样本的值。允许的值是 `avg` 、 `min` 、 `max` 、 `sum` 和 `median` 。默认是 `avg` 。 |

## 参考样例

以下示例返回数据中 `taxful_total_price` 和 `products.base_price` 字段的统计信息：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "matrix_stats_taxful_total_price": {
      "matrix_stats": {
        "fields": ["taxful_total_price", "products.base_price"]
      }
    }
  }
}

```

返回内容包含聚合结果：

```
{
  "took": 250,
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
    "matrix_stats_taxful_total_price": {
      "doc_count": 4675,
      "fields": [
        {
          "name": "products.base_price",
          "count": 4675,
          "mean": 34.99423943014724,
          "variance": 360.5035285833702,
          "skewness": 5.530161335032689,
          "kurtosis": 131.1630632404217,
          "covariance": {
            "products.base_price": 360.5035285833702,
            "taxful_total_price": 846.6489362233169
          },
          "correlation": {
            "products.base_price": 1,
            "taxful_total_price": 0.8444765264325269
          }
        },
        {
          "name": "taxful_total_price",
          "count": 4675,
          "mean": 75.05542864304839,
          "variance": 2788.1879749835425,
          "skewness": 15.812149139923994,
          "kurtosis": 619.1235507385886,
          "covariance": {
            "products.base_price": 846.6489362233169,
            "taxful_total_price": 2788.1879749835425
          },
          "correlation": {
            "products.base_price": 0.8444765264325269,
            "taxful_total_price": 1
          }
        }
      ]
    }
  }
}

```

下表描述了返回内容的字段。
| 统计 | 描述 |
| ---------------- | --------- |
| `count` | 用于聚合的文档数量。 |
| `mean` | 从样本计算得到的字段平均值。 |
| `variance` | 均值偏差的平方，衡量数据分布的离散程度。 |
| `skewness` | 衡量分布相对于均值的不对称性指标。 |
| `kurtosis` | 衡量分布尾部重量的指标。随着尾部变轻，峰度减小。通过评估峰度和偏度来判断一个总体是否可能呈正态分布。 |
| `covariance` | 衡量两个字段之间联合变差的指标。正值表示它们的值朝同一方向变动。 |
| `correlation` | 标准化协方差，用于衡量两个字段之间关系强度的指标。可能的值范围从-1 到 1（包含两端），表示完全负相关到完全正相关。值为 0 表示变量之间没有可识别的关系。 |

## 缺省值

要定义如何处理缺失值，请使用 `missing` 参数。默认情况下，会忽略缺失值。

例如，创建一个索引，其中文档 1 缺少 `gpa` 和 `class_grades` 字段：

```
POST _bulk
{ "create": { "_index": "students", "_id": "1" } }
{ "name": "John Doe" }
{ "create": { "_index": "students", "_id": "2" } }
{ "name": "Jonathan Powers", "gpa": 3.85, "class_grades": [3.0, 3.9, 4.0] }
{ "create": { "_index": "students", "_id": "3" } }
{ "name": "Jane Doe", "gpa": 3.52, "class_grades": [3.2, 2.1, 3.8] }
```

首先，运行不带 `missing` 参数的 `matrix_stats` 聚合：

```
GET students/_search
{
  "size": 0,
  "aggs": {
    "matrix_stats_taxful_total_price": {
      "matrix_stats": {
        "fields": [
          "gpa",
          "class_grades"
        ],
        "mode": "avg"
      }
    }
  }
}
```

在计算矩阵统计时忽略缺失值：

```
{
  "took": 5,
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
    "matrix_stats_taxful_total_price": {
      "doc_count": 2,
      "fields": [
        {
          "name": "gpa",
          "count": 2,
          "mean": 3.684999942779541,
          "variance": 0.05444997482300096,
          "skewness": 0,
          "kurtosis": 1,
          "covariance": {
            "gpa": 0.05444997482300096,
            "class_grades": 0.09899998760223136
          },
          "correlation": {
            "gpa": 1,
            "class_grades": 0.9999999999999991
          }
        },
        {
          "name": "class_grades",
          "count": 2,
          "mean": 3.333333333333333,
          "variance": 0.1800000381469746,
          "skewness": 0,
          "kurtosis": 1,
          "covariance": {
            "gpa": 0.09899998760223136,
            "class_grades": 0.1800000381469746
          },
          "correlation": {
            "gpa": 0.9999999999999991,
            "class_grades": 1
          }
        }
      ]
    }
  }
}

```

要设置缺失字段为 0 ，请将 `missing` 参数作为键值映射提供。尽管 `class_grades` 是数组字段，但 `matrix_stats` 聚合会将多值数字字段展平为每个文档的平均值，因此您必须将单个数字作为缺失值提供：

```
GET students/_search
{
  "size": 0,
  "aggs": {
    "matrix_stats_taxful_total_price": {
      "matrix_stats": {
        "fields": ["gpa", "class_grades"],
        "mode": "avg",
        "missing": {
          "gpa": 0,
          "class_grades": 0
        }
      }
    }
  }
}
```

在计算矩阵统计时，会用 0 替换任何缺失的 `gpa` 或 `class_grades` 值：

```
{
  "took": 23,
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
    "matrix_stats_taxful_total_price": {
      "doc_count": 3,
      "fields": [
        {
          "name": "gpa",
          "count": 3,
          "mean": 2.456666628519694,
          "variance": 4.55363318017324,
          "skewness": -0.688130006360758,
          "kurtosis": 1.5,
          "covariance": {
            "gpa": 4.55363318017324,
            "class_grades": 4.143944374667273
          },
          "correlation": {
            "gpa": 1,
            "class_grades": 0.9970184390038257
          }
        },
        {
          "name": "class_grades",
          "count": 3,
          "mean": 2.2222222222222223,
          "variance": 3.793703722777191,
          "skewness": -0.6323693521730989,
          "kurtosis": 1.5000000000000002,
          "covariance": {
            "gpa": 4.143944374667273,
            "class_grades": 3.793703722777191
          },
          "correlation": {
            "gpa": 0.9970184390038257,
            "class_grades": 1
          }
        }
      ]
    }
  }
}
```

