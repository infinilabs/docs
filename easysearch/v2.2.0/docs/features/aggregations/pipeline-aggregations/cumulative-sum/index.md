---
title: "累积和聚合（Cumulative Sum）"
date: 0001-01-01
summary: "累积和聚合 #  cumulative_sum 聚合是一个父聚合，用于计算上一个聚合的存储分组的累积总和。
累积和是给定序列的部分和的序列。例如，序列 {a, b, c, ...} 的累积和为 a、a+b、a+b+c 等。您可以使用累积总和来可视化字段随时间的变化率。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  cumulative_sum 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 String 要聚合的聚合存储分组的路径。请参阅存储分组路径 。   format 可选 String DecimalFormat 格式字符串。返回聚合的 value_as_string 属性中的格式化输出。    参考样例 #  以下示例创建间隔为一个月的日期直方图。sum 子聚合计算每个月所有字节的总和。最后，cumulative_sum 聚合计算每个月存储分组的累积字节数：
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;sales_per_month&#34;: { &#34;date_histogram&#34;: { &#34;field&#34;: &#34;@timestamp&#34;, &#34;calendar_interval&#34;: &#34;month&#34; }, &#34;aggs&#34;: { &#34;no-of-bytes&#34;: { &#34;sum&#34;: { &#34;field&#34;: &#34;bytes&#34; } }, &#34;cumulative_bytes&#34;: { &#34;cumulative_sum&#34;: { &#34;buckets_path&#34;: &#34;no-of-bytes&#34; } } } } } } 返回内容"
---


# 累积和聚合

`cumulative_sum` 聚合是一个父聚合，用于计算上一个聚合的存储分组的累积总和。

累积和是给定序列的部分和的序列。例如，序列 `{a, b, c, ...}` 的累积和为 `a`、`a+b`、`a+b+c` 等。您可以使用累积总和来可视化字段随时间的变化率。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`cumulative_sum` 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型 | 描述                                                                        |
| -------------- | --------- | -------- | --------------------------------------------------------------------------- |
| `buckets_path` | 必需      | String   | 要聚合的聚合存储分组的路径。请参阅存储分组路径 。                           |
| `format`       | 可选      | String   | DecimalFormat 格式字符串。返回聚合的 `value_as_string` 属性中的格式化输出。 |

## 参考样例

以下示例创建间隔为一个月的日期直方图。`sum` 子聚合计算每个月所有字节的总和。最后，`cumulative_sum` 聚合计算每个月存储分组的累积字节数：

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
        "no-of-bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "cumulative_bytes": {
          "cumulative_sum": {
            "buckets_path": "no-of-bytes"
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
  "took": 8,
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
          "no-of-bytes": {
            "value": 2804103
          },
          "cumulative_bytes": {
            "value": 2804103
          }
        },
        {
          "key_as_string": "2025-04-01T00:00:00.000Z",
          "key": 1743465600000,
          "doc_count": 6849,
          "no-of-bytes": {
            "value": 39103067
          },
          "cumulative_bytes": {
            "value": 41907170
          }
        },
        {
          "key_as_string": "2025-05-01T00:00:00.000Z",
          "key": 1746057600000,
          "doc_count": 6745,
          "no-of-bytes": {
            "value": 37818519
          },
          "cumulative_bytes": {
            "value": 79725689
          }
        }
      ]
    }
  }
}
```

