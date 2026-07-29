---
title: "桶排序聚合（Bucket Sort）"
date: 0001-01-01
summary: "桶排序聚合 #  bucket_sort 聚合是一个父聚合，它对其父多存储分组聚合生成的存储分组进行排序或截断。
在 bucket_sort 聚合中，您可以按多个字段对存储分组进行排序，每个字段都有自己的排序顺序。可以按存储分组的键、文档计数或子聚合中的值进行排序。您还可以使用 from 和 size 参数来截断结果，无论是否进行排序。
有关指定排序顺序的信息，请参阅排序结果。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  bucket_sort 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     gap_policy 可选 String 要应用于缺失数据的策略。有效值为 skip 和 insert_zeros。默认值为 skip。请参阅数据差距 。   sort 可选 String 要排序的字段列表。请参阅排序结果 。   from 可选 String 要返回的第一个结果的索引。必须是非负整数。默认值为 0。请参阅 from 和 size 参数 。   size 可选 String 要返回的最大结果数。必须是正整数。请参阅 from 和 size 参数 。     您必须至少提供一个 sort、from 和 size。"
---


# 桶排序聚合

`bucket_sort` 聚合是一个父聚合，它对其父多存储分组聚合生成的存储分组进行排序或截断。

在 `bucket_sort` 聚合中，您可以按多个字段对存储分组进行排序，每个字段都有自己的排序顺序。可以按存储分组的键、文档计数或子聚合中的值进行排序。您还可以使用 `from` 和 `size` 参数来截断结果，无论是否进行排序。

有关指定排序顺序的信息，请参阅排序结果。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`bucket_sort` 聚合采用以下参数。

| 参数         | 必需/可选 | 数据类型 | 描述                                                                                          |
| ------------ | --------- | -------- | --------------------------------------------------------------------------------------------- |
| `gap_policy` | 可选      | String   | 要应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros`。默认值为 `skip`。请参阅数据差距 。 |
| `sort`       | 可选      | String   | 要排序的字段列表。请参阅排序结果 。                                                           |
| `from`       | 可选      | String   | 要返回的第一个结果的索引。必须是非负整数。默认值为 0。请参阅 `from` 和 `size` 参数 。         |
| `size`       | 可选      | String   | 要返回的最大结果数。必须是正整数。请参阅 from 和 size 参数 。                                 |

> 您必须至少提供一个 sort、from 和 size。

## 参考样例

以下示例创建间隔为一个月的日期直方图。sum 子聚合计算每个月所有字节的总和。最后，聚合按字节数降序对存储分组进行排序：

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
        "total_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "bytes_bucket_sort": {
          "bucket_sort": {
            "sort": [
              { "total_bytes": { "order": "desc" } }
            ]
          }
        }
      }
    }
  }
}

```

## 返回内容

聚合按总字节数降序对存储分组重新排序：

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
    "sales_per_month": {
      "buckets": [
        {
          "key_as_string": "2025-05-01T00:00:00.000Z",
          "key": 1746057600000,
          "doc_count": 7072,
          "total_bytes": {
            "value": 40124337
          }
        },
        {
          "key_as_string": "2025-06-01T00:00:00.000Z",
          "key": 1748736000000,
          "doc_count": 6056,
          "total_bytes": {
            "value": 34123131
          }
        },
        {
          "key_as_string": "2025-04-01T00:00:00.000Z",
          "key": 1743465600000,
          "doc_count": 946,
          "total_bytes": {
            "value": 5478221
          }
        }
      ]
    }
  }
}

```

## 示例：截取结果

要截取结果，请提供 from 和/或 size 参数。以下示例执行相同的排序，但返回两个存储分组，从第二个存储分组开始：

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
        "total_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "bytes_bucket_sort": {
          "bucket_sort": {
            "sort": [
              { "total_bytes": { "order": "desc" } }
            ],
            "from": 1,
            "size": 2
          }
        }
      }
    }
  }
}

```

聚合返回两个排序的存储分组：

```
{
  "took": 2,
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
          "key_as_string": "2025-06-01T00:00:00.000Z",
          "key": 1748736000000,
          "doc_count": 6056,
          "total_bytes": {
            "value": 34123131
          }
        },
        {
          "key_as_string": "2025-04-01T00:00:00.000Z",
          "key": 1743465600000,
          "doc_count": 946,
          "total_bytes": {
            "value": 5478221
          }
        }
      ]
    }
  }
}

```

要截取结果而不进行排序，请省略 `sort` 参数：

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
        "total_bytes": {
          "sum": {
            "field": "bytes"
          }
        },
        "bytes_bucket_sort": {
          "bucket_sort": {
            "from": 1,
            "size": 2
          }
        }
      }
    }
  }
}

```

