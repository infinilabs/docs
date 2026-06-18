---
title: "平均桶聚合（Avg Bucket）"
date: 0001-01-01
summary: "平均桶聚合 #  avg_bucket 聚合是一个同级聚合，它计算上一个聚合的每个分组中的指标平均值。
指定的指标必须是数值型的，且同级聚合必须是多分组聚合。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  avg_bucket 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 String 要聚合的聚合存储分组的路径。请参阅存储分组路径 。   gap_policy 可选 String 要应用于缺失数据的策略。有效值为 skip 和 insert_zeros。默认值为 skip。请参阅数据差距 。   format 可选 String DecimalFormat 格式字符串。返回聚合的 value_as_string 属性中的格式化输出 。    参考样例 #  以下示例创建间隔为一个月的日期直方图。sum 子聚合计算每个月的字节总和。最后，avg_bucket 聚合根据这些总和计算每月的平均字节数：
POST sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;visits_per_month&#34;: { &#34;date_histogram&#34;: { &#34;field&#34;: &#34;@timestamp&#34;, &#34;interval&#34;: &#34;month&#34; }, &#34;aggs&#34;: { &#34;sum_of_bytes&#34;: { &#34;sum&#34;: { &#34;field&#34;: &#34;bytes&#34; } } } }, &#34;avg_monthly_bytes&#34;: { &#34;avg_bucket&#34;: { &#34;buckets_path&#34;: &#34;visits_per_month&gt;sum_of_bytes&#34; } } } } 返回内容 #  聚合返回每月存储分组的平均字节数："
---


# 平均桶聚合

`avg_bucket` 聚合是一个同级聚合，它计算上一个聚合的每个分组中的指标平均值。

指定的指标必须是数值型的，且同级聚合必须是多分组聚合。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

avg_bucket 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型 | 描述                                                                                          |
| -------------- | --------- | -------- | --------------------------------------------------------------------------------------------- |
| `buckets_path` | 必需      | String   | 要聚合的聚合存储分组的路径。请参阅存储分组路径 。                                             |
| `gap_policy`   | 可选      | String   | 要应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros`。默认值为 `skip`。请参阅数据差距 。 |
| `format`       | 可选      | String   | DecimalFormat 格式字符串。返回聚合的 `value_as_string` 属性中的格式化输出 。                  |

## 参考样例

以下示例创建间隔为一个月的日期直方图。`sum` 子聚合计算每个月的字节总和。最后，`avg_bucket` 聚合根据这些总和计算每月的平均字节数：

```
POST sample_data_logs/_search
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
    "avg_monthly_bytes": {
      "avg_bucket": {
        "buckets_path": "visits_per_month>sum_of_bytes"
      }
    }
  }
}
```

## 返回内容

聚合返回每月存储分组的平均字节数：

```
{
  "took": 43,
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
    "avg_monthly_bytes": {
      "value": 26575229.666666668
    }
  }
}
```

