---
title: "导数聚合（Derivative）"
date: 0001-01-01
summary: "导数聚合 #  derivative 聚合是一个父聚合，用于计算聚合每个分组的一阶和二阶导数。
对于有序的分组序列，derivative 将当前分组和前一个分组中的指标值之差近似为一阶导数。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  derivative 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 String 要聚合的聚合分组的路径。参见分组路径。   gap_policy 可选 String 应用于缺失数据的策略。有效值为 skip 和 insert_zeros 。默认为 skip 。参见数据间隙。   format 可选 String DecimalFormat 格式字符串。返回聚合的 value_as_string 属性中的格式化输出。    示例：一阶导数 #  以下示例创建一个每月间隔的日期直方图。 sum 子聚合计算每个月所有字节的和。最后， derivative 聚合计算 sum 子聚合的一阶导数。一阶导数估计为当前月份和上个月字节数之间的差值："
---


# 导数聚合

`derivative` 聚合是一个父聚合，用于计算聚合每个分组的一阶和二阶导数。

对于有序的分组序列，`derivative` 将当前分组和前一个分组中的指标值之差近似为一阶导数。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`derivative` 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型 | 描述                                                                                     |
| -------------- | --------- | -------- | ---------------------------------------------------------------------------------------- |
| `buckets_path` | 必需      | String   | 要聚合的聚合分组的路径。参见分组路径。                                                   |
| `gap_policy`   | 可选      | String   | 应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros` 。默认为 `skip` 。参见数据间隙。 |
| `format`       | 可选      | String   | DecimalFormat 格式字符串。返回聚合的 `value_as_string` 属性中的格式化输出。              |

## 示例：一阶导数

以下示例创建一个每月间隔的日期直方图。 `sum` 子聚合计算每个月所有字节的和。最后， `derivative` 聚合计算 `sum` 子聚合的一阶导数。一阶导数估计为当前月份和上个月字节数之间的差值：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "number_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "bytes_deriv": {
          "derivative": {
            "buckets_path": "number_of_bytes"
          }
        }
      }
    }
  }
}
```

返回内容显示了为第二和第三个分组计算出的导数：

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
    "sales_per_month": {
      "buckets": [
        {
          "key_as_string": "2025-03-01T00:00:00.000Z",
          "key": 1740787200000,
          "doc_count": 480,
          "number_of_bytes": {
            "value": 2804103
          }
        },
        {
          "key_as_string": "2025-04-01T00:00:00.000Z",
          "key": 1743465600000,
          "doc_count": 6849,
          "number_of_bytes": {
            "value": 39103067
          },
          "bytes_deriv": {
            "value": 36298964
          }
        },
        {
          "key_as_string": "2025-05-01T00:00:00.000Z",
          "key": 1746057600000,
          "doc_count": 6745,
          "number_of_bytes": {
            "value": 37818519
          },
          "bytes_deriv": {
            "value": -1284548
          }
        }
      ]
    }
  }
}
```

第一个分组没有计算导数，因为该分组没有可用的前一个分组。

## 示例：二阶导数

要计算二阶导数，将一个导数聚合连接到另一个导数聚合上：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "number_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "bytes_1st_deriv": {
          "derivative": {
            "buckets_path": "number_of_bytes"
          }
        },
        "bytes_2nd_deriv": {
          "derivative": {
            "buckets_path": "bytes_1st_deriv"
          }
        }
      }
    }
  }
}

```

返回内容

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
    "sales_per_month": {
      "buckets": [
        {
          "key_as_string": "2025-03-01T00:00:00.000Z",
          "key": 1740787200000,
          "doc_count": 480,
          "number_of_bytes": {
            "value": 2804103
          }
        },
        {
          "key_as_string": "2025-04-01T00:00:00.000Z",
          "key": 1743465600000,
          "doc_count": 6849,
          "number_of_bytes": {
            "value": 39103067
          },
          "bytes_1st_deriv": {
            "value": 36298964
          }
        },
        {
          "key_as_string": "2025-05-01T00:00:00.000Z",
          "key": 1746057600000,
          "doc_count": 6745,
          "number_of_bytes": {
            "value": 37818519
          },
          "bytes_1st_deriv": {
            "value": -1284548
          },
          "bytes_2nd_deriv": {
            "value": -37583512
          }
        }
      ]
    }
  }
}

```

由于该分组没有可用的前一个分组，因此第一个分组没有计算一阶导数。类似地，第一个和第二个分组也没有计算二阶导数。

