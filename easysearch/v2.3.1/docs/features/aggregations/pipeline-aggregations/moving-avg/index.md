---
title: "移动平均聚合（Moving Avg）"
date: 0001-01-01
summary: "移动平均聚合 #  moving_avg 聚合是一个父级管道聚合，它计算有序数据集中窗口（相邻子集）内指标的一系列平均值。
要创建一个 moving_avg 聚合，您首先创建一个 histogram 或 date_histogram 聚合。然后，您可以选择在直方图聚合中嵌入一个指标聚合。最后，您在直方图中嵌入 moving_avg 聚合，并将 buckets_path 参数设置为要跟踪的嵌入指标。
相关指南（先读这些） #    聚合基础  聚合场景实践  窗口的大小是窗口中连续数据值的数量。在每次迭代中，算法计算窗口中所有数据点的平均值，然后向前滑动一个数据值，排除上一个窗口的第一个值，并包含下一个窗口的第一个值。
例如，给定数据 [1, 5, 8, 23, 34, 28, 7, 23, 20, 19] ，一个窗口大小为 5 的移动平均如下：
(1 + 5 + 8 + 23 + 34) / 5 = 14.2 (5 + 8 + 23 + 34 + 28) / 5 = 19.6 (8 + 23 + 34 + 28 + 7) / 5 = 20 ."
---


# 移动平均聚合

`moving_avg` 聚合是一个父级管道聚合，它计算有序数据集中窗口（相邻子集）内指标的一系列平均值。

要创建一个 `moving_avg` 聚合，您首先创建一个 `histogram` 或 `date_histogram` 聚合。然后，您可以选择在直方图聚合中嵌入一个指标聚合。最后，您在直方图中嵌入 `moving_avg` 聚合，并将 `buckets_path` 参数设置为要跟踪的嵌入指标。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

窗口的大小是窗口中连续数据值的数量。在每次迭代中，算法计算窗口中所有数据点的平均值，然后向前滑动一个数据值，排除上一个窗口的第一个值，并包含下一个窗口的第一个值。

例如，给定数据 `[1, 5, 8, 23, 34, 28, 7, 23, 20, 19]` ，一个窗口大小为 5 的移动平均如下：

```
(1 + 5 + 8 + 23 + 34) / 5 = 14.2
(5 + 8 + 23 + 34 + 28) / 5 = 19.6
(8 + 23 + 34 + 28 + 7) / 5 = 20
...
```

`moving_avg` 聚合通常应用于时间序列数据，以平滑噪声或短期波动，并识别趋势。指定较小的窗口大小以平滑小规模波动。指定较大的窗口大小以平滑高频波动或随机噪声，使低频趋势更加明显。

## 参数说明

`moving_avg` 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型  | 描述                                                                                                                       |
| -------------- | --------- | --------- | -------------------------------------------------------------------------------------------------------------------------- |
| `buckets_path` | 必需      | String    | 要聚合的聚合分组的路径。参见分组路径。                                                                                     |
| `gap_policy`   | 可选      | String    | 应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros` 。默认为 `skip` 。参见数据间隙。                                   |
| `format`       | 可选      | String    | DecimalFormat 格式字符串。返回聚合的 `value_as _string` 属性中的格式化输出。                                               |
| `window`       | 可选      | Numerical | 窗口中包含的数据点数量。默认为 5 。                                                                                        |
| `model`        | 可选      | String    | 要使用的加权移动平均模型。选项为 `ewma` 、 `holt` 、 `holt_winters` 、 `linear` 和 `simple` 。默认为 `simple` 。参见模型。 |
| `settings`     | 可选      | Object    | 调整窗口的参数。参见模型。                                                                                                 |
| `predict`      | 可选      | Numerical | 要追加到结果末尾的预测值数量。默认为 0 。                                                                                  |

## 参考样例

以下示例创建一个日期直方图，间隔为一个月。 `sum` 子聚合计算每个月的字节总和。最后， `moving_avg` 聚合计算这些总和的每月移动平均值：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "my_date_histogram": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": { "field": "bytes" }
        },
        "moving_avg_of_sum_of_bytes": {
          "moving_avg": {
            "buckets_path": "sum_of_bytes"
          }
        }
      }
    }
  }
}
```

该聚合从第二个分组开始返回 `moving_avg` 值。第一个分组没有移动平均值，因为没有足够的前置数据点来计算它：

```
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10000,
      "relation": "gte"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "my_date_histogram": {
      "buckets": [
        {
          "key_as_string": "2025-03-01T00:00:00.000Z",
          "key": 1740787200000,
          "doc_count": 480,
          "sum_of_bytes": {
            "value": 2804103
          }
        },
        {
          "key_as_string": "2025-04-01T00:00:00.000Z",
          "key": 1743465600000,
          "doc_count": 6849,
          "sum_of_bytes": {
            "value": 39103067
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 2804103
          }
        },
        {
          "key_as_string": "2025-05-01T00:00:00.000Z",
          "key": 1746057600000,
          "doc_count": 6745,
          "sum_of_bytes": {
            "value": 37818519
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 20953585
          }
        }
      ]
    }
  }
}

```

## 示例：预测数据

你可以使用 `moving_avg` 聚合来预测未来的分组。

以下示例将上一个示例的间隔减少到一周，并在响应的末尾附加了五个预测的一周分组：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "my_date_histogram": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "week"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "moving_avg_of_sum_of_bytes": {
          "moving_avg": {
            "buckets_path": "sum_of_bytes",
            "predict": 5
          }
        }
      }
    }
  }
}

```

返回内容包含五个预测。请注意，预测的分组的 `doc_count` 是 0 :

```
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10000,
      "relation": "gte"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "my_date_histogram": {
      "buckets": [
        {
          "key_as_string": "2025-03-24T00:00:00.000Z",
          "key": 1742774400000,
          "doc_count": 249,
          "sum_of_bytes": {
            "value": 1531493
          }
        },
        {
          "key_as_string": "2025-03-31T00:00:00.000Z",
          "key": 1743379200000,
          "doc_count": 1617,
          "sum_of_bytes": {
            "value": 9213161
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 1531493
          }
        },
        {
          "key_as_string": "2025-04-07T00:00:00.000Z",
          "key": 1743984000000,
          "doc_count": 1610,
          "sum_of_bytes": {
            "value": 9188671
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 5372327
          }
        },
        {
          "key_as_string": "2025-04-14T00:00:00.000Z",
          "key": 1744588800000,
          "doc_count": 1610,
          "sum_of_bytes": {
            "value": 9244851
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 6644441.666666667
          }
        },
        {
          "key_as_string": "2025-04-21T00:00:00.000Z",
          "key": 1745193600000,
          "doc_count": 1609,
          "sum_of_bytes": {
            "value": 9061045
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 7294544
          }
        },
        {
          "key_as_string": "2025-04-28T00:00:00.000Z",
          "key": 1745798400000,
          "doc_count": 1554,
          "sum_of_bytes": {
            "value": 8713507
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 7647844.2
          }
        },
        {
          "key_as_string": "2025-05-05T00:00:00.000Z",
          "key": 1746403200000,
          "doc_count": 1710,
          "sum_of_bytes": {
            "value": 9544718
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 9084247
          }
        },
        {
          "key_as_string": "2025-05-12T00:00:00.000Z",
          "key": 1747008000000,
          "doc_count": 1610,
          "sum_of_bytes": {
            "value": 9155820
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 9150558.4
          }
        },
        {
          "key_as_string": "2025-05-19T00:00:00.000Z",
          "key": 1747612800000,
          "doc_count": 1610,
          "sum_of_bytes": {
            "value": 9025078
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 9143988.2
          }
        },
        {
          "key_as_string": "2025-05-26T00:00:00.000Z",
          "key": 1748217600000,
          "doc_count": 895,
          "sum_of_bytes": {
            "value": 5047345
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 9100033.6
          }
        },
        {
          "key_as_string": "2025-06-02T00:00:00.000Z",
          "key": 1748822400000,
          "doc_count": 0,
          "moving_avg_of_sum_of_bytes": {
            "value": 8297293.6
          }
        },
        {
          "key_as_string": "2025-06-09T00:00:00.000Z",
          "key": 1749427200000,
          "doc_count": 0,
          "moving_avg_of_sum_of_bytes": {
            "value": 8297293.6
          }
        },
        {
          "key_as_string": "2025-06-16T00:00:00.000Z",
          "key": 1750032000000,
          "doc_count": 0,
          "moving_avg_of_sum_of_bytes": {
            "value": 8297293.6
          }
        },
        {
          "key_as_string": "2025-06-23T00:00:00.000Z",
          "key": 1750636800000,
          "doc_count": 0,
          "moving_avg_of_sum_of_bytes": {
            "value": 8297293.6
          }
        },
        {
          "key_as_string": "2025-06-30T00:00:00.000Z",
          "key": 1751241600000,
          "doc_count": 0,
          "moving_avg_of_sum_of_bytes": {
            "value": 8297293.6
          }
        }
      ]
    }
  }
}
```

## 模型

`moving_avg` 聚合支持五种模型，这些模型在如何对移动窗口中的值进行加权方面有所不同。

使用 `model` 参数来指定要使用的模型。

| 模型名称         | 模型关键词     | 加权方式                                   |
| ---------------- | -------------- | ------------------------------------------ |
| 简单             | `simple`       | 窗口中所有值的未加权平均值。               |
| 线性             | `linear`       | 使用线性权重衰减，更重视近期值。           |
| 指数加权移动平均 | `ewma`         | 使用指数递减权重，更重视近期值。           |
| Holt             | `holt`         | 使用第二个指数项来平滑长期趋势。           |
| Holt-Winters     | `holt_winters` | 使用第三个指数项来平滑周期（季节性）效应。 |

可以使用 `settings` 对象来设置模型的属性。下表显示了每个模型的可用设置。

| 模型           | 参数   | 允许的值      | 默认值 | 描述                                                                    |
| -------------- | ------ | ------------- | ------ | ----------------------------------------------------------------------- |
| `simple`       | 无     | Numeric array | 无     | 窗口中所有值的算术平均值。                                              |
| `linear`       | 无     | Numeric array | 无     | 窗口中所有值的加权平均值，较新的值权重更大。                            |
| `ewma`         | alpha  | [0, 1]        | 0.3    | 衰减参数。更高的值会给最近的数据点赋予更大的权重。                      |
| `holt`         | alpha  | [0, 1]        | 0.3    | 用于级别组件的衰减参数。                                                |
|                | beta   | [0, 1]        | 0.1    | 趋势成分的衰减参数。                                                    |
| `holt_winters` | alpha  | [0, 1]        | 0.3    | 水平成分的衰减参数。                                                    |
|                | beta   | [0, 1]        | 0.3    | 趋势成分的衰减参数。                                                    |
|                | gamma  | [0, 1]        | 0.3    | 季节成分的衰减参数。                                                    |
|                | type   | `add`, `mult` | `add`  | 定义季节性建模方式：加性或乘性。                                        |
|                | period | Integer       | 1      | 构成周期的分组的数量。                                                  |
|                | pad    | Boolean       | true   | 是否为 `mult` 类型模型中的 0 值添加一个小的偏移量，以避免除以零的错误。 |

### 示例：Holt 模型

`holt` 模型使用 `alpha` 和 `beta` 参数控制的指数衰减计算权重。

以下请求使用 `window` 大小为 6 、 `alpha` 值为 0.4 、 `beta` 值为 0.2 的 `Holt` 模型计算总每周字节数的移动平均值：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "my_date_histogram": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "week"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "moving_avg_of_sum_of_bytes": {
          "moving_avg": {
            "buckets_path": "sum_of_bytes",
            "window": 6,
            "model": "holt",
            "settings": { "alpha": 0.4, "beta": 0.2 }
          }
        }
      }
    }
  }
}

```

移动平均数从第二个分组开始：

```
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 10000,
      "relation": "gte"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "my_date_histogram": {
      "buckets": [
        {
          "key_as_string": "2025-03-24T00:00:00.000Z",
          "key": 1742774400000,
          "doc_count": 249,
          "sum_of_bytes": {
            "value": 1531493
          }
        },
        {
          "key_as_string": "2025-03-31T00:00:00.000Z",
          "key": 1743379200000,
          "doc_count": 1617,
          "sum_of_bytes": {
            "value": 9213161
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 1531493
          }
        },
        {
          "key_as_string": "2025-04-07T00:00:00.000Z",
          "key": 1743984000000,
          "doc_count": 1610,
          "sum_of_bytes": {
            "value": 9188671
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 4604160.2
          }
        },
        {
          "key_as_string": "2025-04-14T00:00:00.000Z",
          "key": 1744588800000,
          "doc_count": 1610,
          "sum_of_bytes": {
            "value": 9244851
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 6806684.584000001
          }
        },
        {
          "key_as_string": "2025-04-21T00:00:00.000Z",
          "key": 1745193600000,
          "doc_count": 1609,
          "sum_of_bytes": {
            "value": 9061045
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 8341230.127680001
          }
        },
        {
          "key_as_string": "2025-04-28T00:00:00.000Z",
          "key": 1745798400000,
          "doc_count": 1554,
          "sum_of_bytes": {
            "value": 8713507
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 9260724.7236736
          }
        },
        {
          "key_as_string": "2025-05-05T00:00:00.000Z",
          "key": 1746403200000,
          "doc_count": 1710,
          "sum_of_bytes": {
            "value": 9544718
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 9657431.903375873
          }
        },
        {
          "key_as_string": "2025-05-12T00:00:00.000Z",
          "key": 1747008000000,
          "doc_count": 1610,
          "sum_of_bytes": {
            "value": 9155820
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 9173999.55240704
          }
        },
        {
          "key_as_string": "2025-05-19T00:00:00.000Z",
          "key": 1747612800000,
          "doc_count": 1610,
          "sum_of_bytes": {
            "value": 9025078
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 9172040.511275519
          }
        },
        {
          "key_as_string": "2025-05-26T00:00:00.000Z",
          "key": 1748217600000,
          "doc_count": 895,
          "sum_of_bytes": {
            "value": 5047345
          },
          "moving_avg_of_sum_of_bytes": {
            "value": 9108804.964619776
          }
        }
      ]
    }
  }
}
```

