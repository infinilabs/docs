---
title: "移动函数聚合（Moving Function）"
date: 0001-01-01
summary: "移动函数聚合 #  moving_fn 聚合是一个父级管道聚合，它在滑动窗口上执行脚本。滑动窗口在从父级 histogram 或 date_histogram 聚合中提取的一系列值上移动。窗口一次向右移动一个分组；moving_fn 每次窗口移动时都会运行脚本。
使用 moving_fn 聚合在滑动窗口内的数据上执行任何数值计算。你可以使用 moving_fn 用于以下目的：
 趋势分析 异常值检测 自定义时间序列分析 自定义平滑算法 数字信号处理 (DSP)  相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  moving_fn 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 String 要聚合的聚合分组的路径。参见分组路径。   script 必需 String 或 Object 为每个数据窗口计算值的脚本。可以是内联脚本、存储脚本或脚本文件。该脚本可以访问在 buckets_path 参数中定义的变量名。   window 必需 Integer 滑动窗口中的分组的数量。必须是正整数。   gap_policy 可选 String 应用于缺失数据的策略。有效值为 skip 和 insert_zeros 。默认为 skip 。参见数据间隙。   format 可选 String DecimalFormat 格式字符串。返回聚合的 value_as_string 属性中的格式化输出。   shift 可选 Integer 窗口要移动的分组的数量。可以是正数（向未来的分组右移）或负数（向过去的分组左移）。默认是 0 ，将窗口立即放置在当前分组的左侧。参见移动窗口。    移动函数的工作原理 #  moving_fn 聚合操作在有序分组序列上的滑动窗口上。从父聚合中的第一个分组开始， moving_fn 执行以下操作："
---


# 移动函数聚合

`moving_fn` 聚合是一个父级管道聚合，它在滑动窗口上执行脚本。滑动窗口在从父级 `histogram` 或 `date_histogram` 聚合中提取的一系列值上移动。窗口一次向右移动一个分组；`moving_fn` 每次窗口移动时都会运行脚本。

使用 `moving_fn` 聚合在滑动窗口内的数据上执行任何数值计算。你可以使用 `moving_fn` 用于以下目的：

- 趋势分析
- 异常值检测
- 自定义时间序列分析
- 自定义平滑算法
- 数字信号处理 (DSP)

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`moving_fn` 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型         | 描述                                                                                                                                        |
| -------------- | --------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `buckets_path` | 必需      | String           | 要聚合的聚合分组的路径。参见分组路径。                                                                                                      |
| `script`       | 必需      | String 或 Object | 为每个数据窗口计算值的脚本。可以是内联脚本、存储脚本或脚本文件。该脚本可以访问在 `buckets_path` 参数中定义的变量名。                        |
| `window`       | 必需      | Integer          | 滑动窗口中的分组的数量。必须是正整数。                                                                                                      |
| `gap_policy`   | 可选      | String           | 应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros` 。默认为 `skip` 。参见数据间隙。                                                    |
| `format`       | 可选      | String           | DecimalFormat 格式字符串。返回聚合的 `value_as_string` 属性中的格式化输出。                                                                 |
| `shift`        | 可选      | Integer          | 窗口要移动的分组的数量。可以是正数（向未来的分组右移）或负数（向过去的分组左移）。默认是 0 ，将窗口立即放置在当前分组的左侧。参见移动窗口。 |

## 移动函数的工作原理

`moving_fn` 聚合操作在有序分组序列上的滑动窗口上。从父聚合中的第一个分组开始， `moving_fn` 执行以下操作：

1. 收集由 `window` 和 `shift` 参数指定的分组中的子序列（窗口）的值。
2. 将这些值作为数组传递给由 `script` 指定的函数。
3. 使用 `script` 从数组中计算出一个值。
4. 将此值作为当前分组的结果返回。
5. 向前移动一个分组并重复此过程。

> “过去”和“未来”值暗示时间序列数据，这是移动窗口函数最常见的用例。更一般地说，它们分别指任何有序数据序列中的先前值和即将到来的值。

`moving_fn` 应用的脚本可以是一个预定义函数或自定义脚本。分组值通过 `values` 数组提供给脚本。脚本返回一个双精度值作为结果。结果值 `NaN` 和 `+/- Inf` 是允许的，但 `null` 是不允许的。

### 窗口大小

`window` 参数指定定义窗口大小的分组的数量。

传递给 `script` 函数的数组是零索引的。其值在脚本中通过 `values[0]` 到 `values[n]` 访问，其中 `n = values.length - 1` 。

### 移动窗口

`shift` 参数控制移动窗口相对于当前分组的位置。根据您的分析需求设置 `shift` ，需要历史背景、当前数据还是未来预测。默认值是 0 ，仅显示过去值（不包括当前分组）。

`shift` 的常用值如下：

- 0:仅过去值。不包括当前值。
- 1:过去值，包括当前值。
- `window/2`:将窗口围绕当前值居中。
- `window`:未来的值，包括当前值。

当窗口扩展到序列开头或结尾的可用数据之外时， `window` 会自动缩小，仅使用可用点：

## 预定义函数

`moving_fn` 聚合支持多种预定义函数，可用于替代自定义脚本。这些函数可通过 `MovingFunctions` 上下文访问。例如，你可以通过 `MovingFunctions.max(values)` 访问 `max` 函数。

下表描述了预定义函数。

| 函数             | 模型关键词          | 描述                                                       |
| ---------------- | ------------------- | ---------------------------------------------------------- |
| 最大值           | `max`               | 窗口中的最大值。                                           |
| 最小值           | `min`               | 窗口中的最小值。                                           |
| 求和             | `sum`               | 窗口中值的总和。                                           |
| 无权平均         | `unweightedAvg`     | 窗口内所有值的未加权平均值，等于 `sum` / `window` 。       |
| 线性加权平均     | `linearWeightedAvg` | 使用线性衰减权重的加权平均，更重视近期值。                 |
| 指数加权移动平均 | `ewma`              | 使用指数衰减权重的加权平均，更重视近期值。                 |
| Holt             | `holt`              | 使用第二个指数项的加权平均，用于平滑长期趋势。             |
| Holt-Winters     | `holt_wimnters	`     | 使用第三个指数项的加权平均，用于平滑周期性（季节性）效应。 |
| 标准差           | `stdDev`            | 窗口中值的总和。                                           |

所有预定义函数都以 `values` 数组作为其第一个参数。对于需要额外参数的函数，按顺序在 `values` 后传递这些参数。例如，通过将 `script` 值设置为 `MovingFunctions.stdDev(values, MovingFunctions.unweightedAvg(values))` 来调用 `stdDev` 函数。

下表显示了每个模型所需的设置。

| 函数                | 参数   | 参数类型      | 默认值 | 描述                                                                                                              |
| ------------------- | ------ | ------------- | ------ | ----------------------------------------------------------------------------------------------------------------- |
| `max`               | 无     | Numeric array | 无     | 窗口的最大值。                                                                                                    |
| `min`               | 无     | Numeric array | 无     | 窗口的最小值。                                                                                                    |
| `sum`               | 无     | Numeric array | 无     | 窗口中所有值的总和。                                                                                              |
| `unweightedAvg`     | 无     | Numeric array | 无     | 窗口中所有值的算术平均值。                                                                                        |
| `linearWeightedAvg` | 无     | Numeric array | 无     | 窗口中所有值的加权平均值，较新的值权重更大。                                                                      |
| `ewma`              | alpha  | [0, 1]        | 0.3    | 衰减参数。更高的值会给最近的数据点赋予更大的权重。                                                                |
| `holt`              | alpha  | [0, 1]        | 0.3    | 用于级别组件的衰减参数。                                                                                          |
|                     | beta   | [0, 1]        | 0.1    | 趋势成分的衰减参数。                                                                                              |
| `holt_winters`      | alpha  | [0, 1]        | 0.3    | 用于级别组件的衰减参数。                                                                                          |
|                     | beta   | [0, 1]        | 0.3    | 趋势成分的衰减参数。                                                                                              |
|                     | gamma  | [0, 1]        | 0.3    | 季节成分的衰减参数。                                                                                              |
|                     | type   | `add`, `mult` | `add`  | 定义季节性如何建模：加性或乘性。                                                                                  |
|                     | period | Integer       | 1      | 构成周期的分组数。                                                                                                |
|                     | pad    | Boolean       | true   | 是否为 0 类型的模型对 `mult` 值添加一个小的偏移量以避免除以零的错误。                                             |
| `stdDev`            | avg    | double        | 无     | 窗口的标准差。要计算有意义的标准差，请使用滑动窗口数组的平均值，通常为 `MovingFunctions.unweightedAvg(values)` 。 |

> 预定义函数不支持缺少参数的函数签名。因此，即使使用默认值，您也必须提供额外的参数。

### 示例：预定义函数

以下示例创建一个日期直方图，间隔为一周。 `sum` 子聚合计算每周所有记录的字节总和。最后， `moving_fn` 聚合使用 `window` 大小为 5 、默认 `shift` 为 0 ，以及无权重的平均值来计算字节总和的标准差：

```
POST sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "week"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "bytes" }
        },
        "the_movavg": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 5,
            "script": "MovingFunctions.stdDev(values, MovingFunctions.unweightedAvg(values))"
          }
        }
      }
    }
  }
}

```

返回内容显示了移动窗口的标准差，从第二个分组开始为零值。对于空窗口或只包含无效值（ `null` 或 `NaN` ）的窗口， `stdDev` 函数返回 0 。

```
{
  "took": 15,
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
    "my_date_histo": {
      "buckets": [
        {
          "key_as_string": "2025-03-24T00:00:00.000Z",
          "key": 1742774400000,
          "doc_count": 249,
          "the_sum": {
            "value": 1531493
          },
          "the_movavg": {
            "value": null
          }
        },
        {
          "key_as_string": "2025-03-31T00:00:00.000Z",
          "key": 1743379200000,
          "doc_count": 1617,
          "the_sum": {
            "value": 9213161
          },
          "the_movavg": {
            "value": 0
          }
        },
        {
          "key_as_string": "2025-04-07T00:00:00.000Z",
          "key": 1743984000000,
          "doc_count": 1610,
          "the_sum": {
            "value": 9188671
          },
          "the_movavg": {
            "value": 3840834
          }
        },
        {
          "key_as_string": "2025-04-14T00:00:00.000Z",
          "key": 1744588800000,
          "doc_count": 1610,
          "the_sum": {
            "value": 9244851
          },
          "the_movavg": {
            "value": 3615414.498228507
          }
        },
        {
          "key_as_string": "2025-04-21T00:00:00.000Z",
          "key": 1745193600000,
          "doc_count": 1609,
          "the_sum": {
            "value": 9061045
          },
          "the_movavg": {
            "value": 3327358.65618917
          }
        },
        {
          "key_as_string": "2025-04-28T00:00:00.000Z",
          "key": 1745798400000,
          "doc_count": 1554,
          "the_sum": {
            "value": 8713507
          },
          "the_movavg": {
            "value": 3058812.9440705855
          }
        },
        {
          "key_as_string": "2025-05-05T00:00:00.000Z",
          "key": 1746403200000,
          "doc_count": 1710,
          "the_sum": {
            "value": 9544718
          },
          "the_movavg": {
            "value": 195603.33146038183
          }
        },
        {
          "key_as_string": "2025-05-12T00:00:00.000Z",
          "key": 1747008000000,
          "doc_count": 1610,
          "the_sum": {
            "value": 9155820
          },
          "the_movavg": {
            "value": 270085.92336040025
          }
        },
        {
          "key_as_string": "2025-05-19T00:00:00.000Z",
          "key": 1747612800000,
          "doc_count": 1610,
          "the_sum": {
            "value": 9025078
          },
          "the_movavg": {
            "value": 269477.75659701484
          }
        },
        {
          "key_as_string": "2025-05-26T00:00:00.000Z",
          "key": 1748217600000,
          "doc_count": 895,
          "the_sum": {
            "value": 5047345
          },
          "the_movavg": {
            "value": 267356.5422566652
          }
        }
      ]
    }
  }
}
```

## 自定义脚本

你可以提供一个任意的自定义脚本来计算 `moving_fn` 结果。自定义脚本使用 Painless 脚本语言。

### 示例：使用自定义脚本

以下示例创建一个日期直方图，间隔为一周。 `sum` 子聚合计算每周所有应税收入的总额。 `moving_fn` 脚本返回当前值之前的两个值中较大的那个值，如果两个值不可用，则返回 `NaN` 。

```
POST sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "week"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "taxful_total_price" }
        },
        "the_movavg": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 2,
            "script": "return (values.length < 2 ? Double.NaN : (values[0]>values[1] ? values[0] : values[1]))"
          }
        }
      }
    }
  }
}

```

该示例返回从第三个分组开始的计算结果，其中存在足够的前置数据来执行计算：

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
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "my_date_histo": {
      "buckets": [
        {
          "key_as_string": "2025-03-24T00:00:00.000Z",
          "key": 1742774400000,
          "doc_count": 582,
          "the_sum": {
            "value": 41455.5390625
          },
          "the_movavg": {
            "value": null
          }
        },
        {
          "key_as_string": "2025-03-31T00:00:00.000Z",
          "key": 1743379200000,
          "doc_count": 1048,
          "the_sum": {
            "value": 79448.60546875
          },
          "the_movavg": {
            "value": null
          }
        },
        {
          "key_as_string": "2025-04-07T00:00:00.000Z",
          "key": 1743984000000,
          "doc_count": 1048,
          "the_sum": {
            "value": 78208.4296875
          },
          "the_movavg": {
            "value": 79448.60546875
          }
        },
        {
          "key_as_string": "2025-04-14T00:00:00.000Z",
          "key": 1744588800000,
          "doc_count": 1073,
          "the_sum": {
            "value": 81277.296875
          },
          "the_movavg": {
            "value": 79448.60546875
          }
        },
        {
          "key_as_string": "2025-04-21T00:00:00.000Z",
          "key": 1745193600000,
          "doc_count": 924,
          "the_sum": {
            "value": 70494.2578125
          },
          "the_movavg": {
            "value": 81277.296875
          }
        }
      ]
    }
  }
}
```

## 示例：移动平均聚合

`holt` 模型是一种移动平均，它使用由 `alpha` 和 `beta` 参数控制的指数衰减权重。以下示例创建一个日期直方图，间隔为一周。 sum 子聚合计算每周所有字节的总和。最后， `moving_fn` 聚合使用 `Holt` 模型计算字节总和的加权平均值，模型大小为 `window` ，默认值为 `shift` ， `alpha` 值为 0.3 ， `beta` 值为 0.1 ：

```
POST sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "my_date_histogram": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "week"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "bytes" }
        },
        "the_movavg": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 6,
            "script": "MovingFunctions.holt(values, 0.3, 0.1)"
          }
        }
      }
    }
  }
}

```

聚合返回从第二个分组开始的移动 `holt` 平均值：

```
{
  "took": 16,
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
          "the_sum": {
            "value": 1531493
          },
          "the_movavg": {
            "value": null
          }
        },
        {
          "key_as_string": "2025-03-31T00:00:00.000Z",
          "key": 1743379200000,
          "doc_count": 1617,
          "the_sum": {
            "value": 9213161
          },
          "the_movavg": {
            "value": 1531493
          }
        },
        {
          "key_as_string": "2025-04-07T00:00:00.000Z",
          "key": 1743984000000,
          "doc_count": 1610,
          "the_sum": {
            "value": 9188671
          },
          "the_movavg": {
            "value": 3835993.3999999994
          }
        },
        {
          "key_as_string": "2025-04-14T00:00:00.000Z",
          "key": 1744588800000,
          "doc_count": 1610,
          "the_sum": {
            "value": 9244851
          },
          "the_movavg": {
            "value": 5603111.707999999
          }
        },
        {
          "key_as_string": "2025-04-21T00:00:00.000Z",
          "key": 1745193600000,
          "doc_count": 1609,
          "the_sum": {
            "value": 9061045
          },
          "the_movavg": {
            "value": 6964515.302359998
          }
        },
        {
          "key_as_string": "2025-04-28T00:00:00.000Z",
          "key": 1745798400000,
          "doc_count": 1554,
          "the_sum": {
            "value": 8713507
          },
          "the_movavg": {
            "value": 7930766.089341199
          }
        },
        {
          "key_as_string": "2025-05-05T00:00:00.000Z",
          "key": 1746403200000,
          "doc_count": 1710,
          "the_sum": {
            "value": 9544718
          },
          "the_movavg": {
            "value": 8536788.607547803
          }
        },
        {
          "key_as_string": "2025-05-12T00:00:00.000Z",
          "key": 1747008000000,
          "doc_count": 1610,
          "the_sum": {
            "value": 9155820
          },
          "the_movavg": {
            "value": 9172269.837272028
          }
        },
        {
          "key_as_string": "2025-05-19T00:00:00.000Z",
          "key": 1747612800000,
          "doc_count": 1610,
          "the_sum": {
            "value": 9025078
          },
          "the_movavg": {
            "value": 9166173.88436614
          }
        },
        {
          "key_as_string": "2025-05-26T00:00:00.000Z",
          "key": 1748217600000,
          "doc_count": 895,
          "the_sum": {
            "value": 5047345
          },
          "the_movavg": {
            "value": 9123157.830417283
          }
        }
      ]
    }
  }
}
```

