---
title: "序列差分聚合（Serial Diff）"
date: 0001-01-01
summary: "序列差分聚合 #  serial_diff 聚合是一个父级管道聚合，用于计算当前分组中指标值与上一个分组中指标值之间的差值。它将结果存储在当前分组中。
使用 serial_diff 聚合来计算具有指定滞后的时间段之间的变化。lag 参数（一个正整数值）指定要从中减去当前值的哪个先前分组的值。默认的 lag 值是 1，这意味着 serial_diff 从当前分组中的值减去立即前一个分组中的值。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  serial_diff 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 String 要聚合的聚合分组的路径。参见分组路径。   gap_policy 可选 String 应用于缺失数据的策略。有效值为 skip 和 insert_zeros 。默认为 skip 。参见数据间隙。   format 可选 String DecimalFormat 格式字符串。返回聚合的 value_as _string 属性中的格式化输出。   lag 可选 Integer 用于从当前数据分组中减去的历史数据分组。必须是正整数。默认为 1 。    参考样例 #  以下示例创建一个日期直方图，间隔为一个月。 sum 子聚合计算每个月的字节总和。最后， serial_diff 聚合计算这些总和之间的月度字节差异："
---


# 序列差分聚合

`serial_diff` 聚合是一个父级管道聚合，用于计算当前分组中指标值与上一个分组中指标值之间的差值。它将结果存储在当前分组中。

使用 `serial_diff` 聚合来计算具有指定滞后的时间段之间的变化。`lag` 参数（一个正整数值）指定要从中减去当前值的哪个先前分组的值。默认的 `lag` 值是 1，这意味着 `serial_diff` 从当前分组中的值减去立即前一个分组中的值。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

serial_diff 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型 | 描述                                                                                     |
| -------------- | --------- | -------- | ---------------------------------------------------------------------------------------- |
| `buckets_path` | 必需      | String   | 要聚合的聚合分组的路径。参见分组路径。                                                   |
| `gap_policy`   | 可选      | String   | 应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros` 。默认为 `skip` 。参见数据间隙。 |
| `format`       | 可选      | String   | DecimalFormat 格式字符串。返回聚合的 `value_as _string` 属性中的格式化输出。             |
| `lag`          | 可选      | Integer  | 用于从当前数据分组中减去的历史数据分组。必须是正整数。默认为 1 。                        |

## 参考样例

以下示例创建一个日期直方图，间隔为一个月。 `sum` 子聚合计算每个月的字节总和。最后， `serial_diff` 聚合计算这些总和之间的月度字节差异：

```
GET sample_data_logs/_search
{
   "size": 0,
   "aggs": {
      "monthly_bytes": {
         "date_histogram": {
            "field": "@timestamp",
            "calendar_interval": "month"
         },
         "aggs": {
            "total_bytes": {
               "sum": {
                  "field": "bytes"
               }
            },
            "monthly_bytes_change": {
               "serial_diff": {
                  "buckets_path": "total_bytes",
                  "lag": 1
               }
            }
         }
      }
   }
}
```

返回内容包含第二个月和第三个月的月度差异。（第一个月 `serial_diff` 无法计算，因为没有前一个月可以与之比较）：

```
{
  "took": 3,
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
    "monthly_bytes": {
      "buckets": [
        {
          "key_as_string": "2025-03-01T00:00:00.000Z",
          "key": 1740787200000,
          "doc_count": 480,
          "total_bytes": {
            "value": 2804103
          }
        },
        {
          "key_as_string": "2025-04-01T00:00:00.000Z",
          "key": 1743465600000,
          "doc_count": 6849,
          "total_bytes": {
            "value": 39103067
          },
          "monthly_bytes_change": {
            "value": 36298964
          }
        },
        {
          "key_as_string": "2025-05-01T00:00:00.000Z",
          "key": 1746057600000,
          "doc_count": 6745,
          "total_bytes": {
            "value": 37818519
          },
          "monthly_bytes_change": {
            "value": -1284548
          }
        }
      ]
    }
  }
}

```

## 示例：多周期差分

使用更大的 `lag` 值来比较每个分组与过去更早时间发生的分组。以下示例计算每周字节数据的差分，滞后为 4（即每个分组与 4 周前的分组进行比较）。这会消除任何周期为 4 周的波动：

```
GET sample_data_logs/_search
{
   "size": 0,
   "aggs": {
      "monthly_bytes": {
         "date_histogram": {
            "field": "@timestamp",
            "calendar_interval": "week"
         },
         "aggs": {
            "total_bytes": {
               "sum": {
                  "field": "bytes"
               }
            },
            "monthly_bytes_change": {
               "serial_diff": {
                  "buckets_path": "total_bytes",
                  "lag": 4
               }
            }
         }
      }
   }
}

```

返回内容包含每周分组的列表。请注意， `serial_diff` 聚合直到第五个分组才开始，当出现一个 `lag` 为 4 的分组时：

```
{
  "took": 6,
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
    "monthly_bytes": {
      "buckets": [
        {
          "key_as_string": "2025-03-24T00:00:00.000Z",
          "key": 1742774400000,
          "doc_count": 249,
          "total_bytes": {
            "value": 1531493
          }
        },
        {
          "key_as_string": "2025-03-31T00:00:00.000Z",
          "key": 1743379200000,
          "doc_count": 1617,
          "total_bytes": {
            "value": 9213161
          }
        },
        {
          "key_as_string": "2025-04-07T00:00:00.000Z",
          "key": 1743984000000,
          "doc_count": 1610,
          "total_bytes": {
            "value": 9188671
          }
        },
        {
          "key_as_string": "2025-04-14T00:00:00.000Z",
          "key": 1744588800000,
          "doc_count": 1610,
          "total_bytes": {
            "value": 9244851
          }
        },
        {
          "key_as_string": "2025-04-21T00:00:00.000Z",
          "key": 1745193600000,
          "doc_count": 1609,
          "total_bytes": {
            "value": 9061045
          },
          "monthly_bytes_change": {
            "value": 7529552
          }
        },
        {
          "key_as_string": "2025-04-28T00:00:00.000Z",
          "key": 1745798400000,
          "doc_count": 1554,
          "total_bytes": {
            "value": 8713507
          },
          "monthly_bytes_change": {
            "value": -499654
          }
        },
        {
          "key_as_string": "2025-05-05T00:00:00.000Z",
          "key": 1746403200000,
          "doc_count": 1710,
          "total_bytes": {
            "value": 9544718
          },
          "monthly_bytes_change": {
            "value": 356047
          }
        },
        {
          "key_as_string": "2025-05-12T00:00:00.000Z",
          "key": 1747008000000,
          "doc_count": 1610,
          "total_bytes": {
            "value": 9155820
          },
          "monthly_bytes_change": {
            "value": -89031
          }
        },
        {
          "key_as_string": "2025-05-19T00:00:00.000Z",
          "key": 1747612800000,
          "doc_count": 1610,
          "total_bytes": {
            "value": 9025078
          },
          "monthly_bytes_change": {
            "value": -35967
          }
        },
        {
          "key_as_string": "2025-05-26T00:00:00.000Z",
          "key": 1748217600000,
          "doc_count": 895,
          "total_bytes": {
            "value": 5047345
          },
          "monthly_bytes_change": {
            "value": -3666162
          }
        }
      ]
    }
  }
}
```

