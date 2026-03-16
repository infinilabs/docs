---
title: "百分位桶聚合（Percentiles Bucket）"
date: 0001-01-01
summary: "百分位桶聚合 #  percentiles_bucket 聚合是一个同级聚合，用于计算分位数的位置。
percentiles_bucket 聚合精确计算分位数，不使用近似或插值。每个分位数都返回为目标分位数小于或等于的最近值。
percentiles_bucket 聚合需要将整个值列表临时保存在内存中，即使对于大型数据集也是如此。相比之下，percentiles 指标聚合使用更少的内存，但会近似百分比。
指定的指标必须是数值型，并且同级聚合必须是多分组聚合。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  percentiles_bucket聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 String 要聚合的聚合分组的路径。参见分组路径。   gap_policy 可选 String 应用于缺失数据的策略。有效值为 skip 和 insert_zeros 。默认为 skip 。参见数据间隙。   format 可选 String DecimalFormat 格式字符串。返回聚合的 value_as _string 属性中的格式化输出。   percents 可选 List 一个包含任意数量数值百分比值的列表，这些值将被包含在输出中。有效值为 0.0 到 100.0（含）。默认为 [1."
---


# 百分位桶聚合

`percentiles_bucket` 聚合是一个同级聚合，用于计算分位数的位置。

`percentiles_bucket` 聚合精确计算分位数，不使用近似或插值。每个分位数都返回为目标分位数小于或等于的最近值。

`percentiles_bucket` 聚合需要将整个值列表临时保存在内存中，即使对于大型数据集也是如此。相比之下，`percentiles` 指标聚合使用更少的内存，但会近似百分比。

指定的指标必须是数值型，并且同级聚合必须是多分组聚合。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`percentiles_bucket`聚合采用以下参数。
| 参数 | 必需/可选 | 数据类型 | 描述 |
| -------------- | --------- | -------- | ---------------------------------------------------------------------------------------- |
| `buckets_path` | 必需 | String | 要聚合的聚合分组的路径。参见分组路径。 |
| `gap_policy` | 可选 | String | 应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros` 。默认为 `skip` 。参见数据间隙。 |
| `format` | 可选 | String | DecimalFormat 格式字符串。返回聚合的 `value_as _string` 属性中的格式化输出。 |
| `percents` | 可选 | List | 一个包含任意数量数值百分比值的列表，这些值将被包含在输出中。有效值为 0.0 到 100.0（含）。默认为 [1.0, 5.0, 25.0, 50.0, 75.0, 95.0, 99.0] 。 |
| `keyed` | 可选 | Boolean | 是否将输出格式化为字典，而不是键值对对象数组。默认为 true （以键值对格式化输出）。 |

## 参考样例

以下示例创建一个以一周为间隔的日期直方图。 `sum` 子聚合为每周汇总 `taxful_total_price` 。最后， `percentiles_bucket` 聚合计算这些汇总的每周百分位数值：

```
POST sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "weekly_sales": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "week"
      },
      "aggs": {
        "total_price": {
          "sum": {
            "field": "taxful_total_price"
          }
        }
      }
    },
    "percentiles_monthly_sales": {
      "percentiles_bucket": {
        "buckets_path": "weekly_sales>total_price"
      }
    }
  }
}

```

聚合返回每周价格总计的默认百分位数值：

```
{
  "took": 4,
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
    "weekly_sales": {
      "buckets": [
        {
          "key_as_string": "2025-03-24T00:00:00.000Z",
          "key": 1742774400000,
          "doc_count": 582,
          "total_price": {
            "value": 41455.5390625
          }
        },
        {
          "key_as_string": "2025-03-31T00:00:00.000Z",
          "key": 1743379200000,
          "doc_count": 1048,
          "total_price": {
            "value": 79448.60546875
          }
        },
        {
          "key_as_string": "2025-04-07T00:00:00.000Z",
          "key": 1743984000000,
          "doc_count": 1048,
          "total_price": {
            "value": 78208.4296875
          }
        },
        {
          "key_as_string": "2025-04-14T00:00:00.000Z",
          "key": 1744588800000,
          "doc_count": 1073,
          "total_price": {
            "value": 81277.296875
          }
        },
        {
          "key_as_string": "2025-04-21T00:00:00.000Z",
          "key": 1745193600000,
          "doc_count": 924,
          "total_price": {
            "value": 70494.2578125
          }
        }
      ]
    },
    "percentiles_monthly_sales": {
      "values": {
        "1.0": 41455.5390625,
        "5.0": 41455.5390625,
        "25.0": 70494.2578125,
        "50.0": 78208.4296875,
        "75.0": 79448.60546875,
        "95.0": 81277.296875,
        "99.0": 81277.296875
      }
    }
  }
}
```

## 示例：选项修改

下一个示例使用与上一个示例相同的数据计算百分位数，但有以下不同：

- `percents` 参数指定仅计算第 25、50 和 75 个百分位数。
- 使用 `format` 参数追加字符串格式输出。
- 通过将 `keyed` 参数设置为 `false` ，结果以键值对对象形式显示（追加字符串值）。

```
POST sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "weekly_sales": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "week"
      },
      "aggs": {
        "total_price": {
          "sum": {
            "field": "taxful_total_price"
          }
        }
      }
    },
    "percentiles_monthly_sales": {
      "percentiles_bucket": {
        "buckets_path": "weekly_sales>total_price",
        "percents": [25.0, 50.0, 75.0],
        "format": "$#,###.00",
        "keyed": false
      }
    }
  }
}

```

选项修改的输出：

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
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "weekly_sales": {
      "buckets": [
        {
          "key_as_string": "2025-03-24T00:00:00.000Z",
          "key": 1742774400000,
          "doc_count": 582,
          "total_price": {
            "value": 41455.5390625
          }
        },
        {
          "key_as_string": "2025-03-31T00:00:00.000Z",
          "key": 1743379200000,
          "doc_count": 1048,
          "total_price": {
            "value": 79448.60546875
          }
        },
        {
          "key_as_string": "2025-04-07T00:00:00.000Z",
          "key": 1743984000000,
          "doc_count": 1048,
          "total_price": {
            "value": 78208.4296875
          }
        },
        {
          "key_as_string": "2025-04-14T00:00:00.000Z",
          "key": 1744588800000,
          "doc_count": 1073,
          "total_price": {
            "value": 81277.296875
          }
        },
        {
          "key_as_string": "2025-04-21T00:00:00.000Z",
          "key": 1745193600000,
          "doc_count": 924,
          "total_price": {
            "value": 70494.2578125
          }
        }
      ]
    },
    "percentiles_monthly_sales": {
      "values": [
        {
          "key": 25,
          "value": 70494.2578125,
          "25.0_as_string": "$70,494.26"
        },
        {
          "key": 50,
          "value": 78208.4296875,
          "50.0_as_string": "$78,208.43"
        },
        {
          "key": 75,
          "value": 79448.60546875,
          "75.0_as_string": "$79,448.61"
        }
      ]
    }
  }
}
```

