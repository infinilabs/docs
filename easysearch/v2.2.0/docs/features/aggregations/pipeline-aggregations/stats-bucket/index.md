---
title: "统计桶聚合（Stats Bucket）"
date: 0001-01-01
summary: "统计桶聚合 #  stats_bucket 聚合是一个同级聚合，它为先前聚合的分组返回各种统计信息（count、min、max、avg 和 sum）。
指定的指标必须是数值型，并且同级聚合必须是多分组聚合。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  stats_bucket 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 String 要聚合的聚合分组的路径。参见分组路径。   gap_policy 可选 String 应用于缺失数据的策略。有效值为 skip 和 insert_zeros 。默认为 skip 。参见数据间隙。   format 可选 String DecimalFormat 格式字符串。返回聚合的 value_as _string 属性中的格式化输出。    参考样例 #  以下示例创建一个以一个月为间隔的日期直方图。 sum 子聚合计算每个月所有字节的总和。最后， stats_bucket 聚合从这些总和中返回 count 、 avg 、 sum 、 min 和 max 统计信息："
---


# 统计桶聚合

`stats_bucket` 聚合是一个同级聚合，它为先前聚合的分组返回各种统计信息（`count`、`min`、`max`、`avg` 和 `sum`）。

指定的指标必须是数值型，并且同级聚合必须是多分组聚合。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`stats_bucket` 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型 | 描述                                                                                     |
| -------------- | --------- | -------- | ---------------------------------------------------------------------------------------- |
| `buckets_path` | 必需      | String   | 要聚合的聚合分组的路径。参见分组路径。                                                   |
| `gap_policy`   | 可选      | String   | 应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros` 。默认为 `skip` 。参见数据间隙。 |
| `format`       | 可选      | String   | DecimalFormat 格式字符串。返回聚合的 `value_as _string` 属性中的格式化输出。             |

## 参考样例

以下示例创建一个以一个月为间隔的日期直方图。 `sum` 子聚合计算每个月所有字节的总和。最后， `stats_bucket` 聚合从这些总和中返回 `count` 、 `avg` 、 `sum` 、 `min` 和 `max` 统计信息：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "visits_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "interval": "month"
      },
      "aggs": {
        "sum_of_bytes": {
          "sum": {
            "field": "bytes"
          }
        }
      }
    },
    "stats_monthly_bytes": {
      "stats_bucket": {
        "buckets_path": "visits_per_month>sum_of_bytes"
      }
    }
  }
}
```

## 返回内容

该聚合返回每个分组的五个基本统计数据：

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
      "value": 10000,
      "relation": "gte"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "visits_per_month": {
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
          }
        },
        {
          "key_as_string": "2025-05-01T00:00:00.000Z",
          "key": 1746057600000,
          "doc_count": 6745,
          "sum_of_bytes": {
            "value": 37818519
          }
        }
      ]
    },
    "stats_monthly_bytes": {
      "count": 3,
      "min": 2804103,
      "max": 39103067,
      "avg": 26575229.666666668,
      "sum": 79725689
    }
  }
}

```

