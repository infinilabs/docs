---
title: "桶选择器聚合（Bucket Selector）"
date: 0001-01-01
summary: "桶选择器聚合 #  bucket_selector 聚合是一个父管道聚合，它评估脚本以确定直方图（或 date_histogram）聚合返回的存储分组是否应包含在最终结果中。
与创建新值的管道聚合不同，bucket_selector 聚合充当筛选器，根据指定的条件保留或删除整个存储分组。使用此聚合可根据存储分组的计算指标筛选存储分组。
相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  bucket_selector 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 Object 变量名称到分分组指标的映射，用于标识要在脚本中使用的指标。指标必须是数字。请参阅脚本变量 。   script 必需 String 或 Object 要执行的脚本。可以是内联脚本、存储脚本或脚本文件。该脚本可以访问 buckets_path 参数中定义的变量名称。必须返回布尔值。返回 false 的存储分组将从最终输出中删除。   gap_policy 可选 String 应用于缺失数据的策略。有效值为 skip 和 insert_zeros 。默认值为 skip 。详见 数据空缺。    参考样例 #  以下示例创建间隔为一周的日期直方图。sum 子聚合计算每周所有销售额的总和。最后，bucket_selector 聚合会筛选生成的每周存储分组，删除所有总值不超过 75,000 美元的存储分组："
---


# 桶选择器聚合

`bucket_selector` 聚合是一个父管道聚合，它评估脚本以确定直方图（或 `date_histogram`）聚合返回的存储分组是否应包含在最终结果中。

与创建新值的管道聚合不同，`bucket_selector` 聚合充当筛选器，根据指定的条件保留或删除整个存储分组。使用此聚合可根据存储分组的计算指标筛选存储分组。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`bucket_selector` 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型         | 描述                                                                                                                                                            |
| -------------- | --------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `buckets_path` | 必需      | Object           | 变量名称到分分组指标的映射，用于标识要在脚本中使用的指标。指标必须是数字。请参阅脚本变量 。                                                                     |
| `script`       | 必需      | String 或 Object | 要执行的脚本。可以是内联脚本、存储脚本或脚本文件。该脚本可以访问 `buckets_path` 参数中定义的变量名称。必须返回布尔值。返回 false 的存储分组将从最终输出中删除。 |
| `gap_policy`   | 可选      | String           | 应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros` 。默认值为 `skip` 。详见 数据空缺。                                                                     |

## 参考样例

以下示例创建间隔为一周的日期直方图。`sum` 子聚合计算每周所有销售额的总和。最后，`bucket_selector` 聚合会筛选生成的每周存储分组，删除所有总值不超过 75,000 美元的存储分组：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "sales_per_week": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "week"
      },
      "aggs": {
        "weekly_sales": {
          "sum": {
            "field": "taxful_total_price",
            "format": "$#,###.00"
          }
        },
        "avg_vendor_spend": {
          "bucket_selector": {
            "buckets_path": {
              "weekly_sales": "weekly_sales"
            },
            "script": "params.weekly_sales > 75000"
          }
        }
      }
    }
  }
}
```

## 返回内容

聚合返回满足脚本条件的 `sales_per_week` 存储分组：

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
      "value": 4675,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "sales_per_week": {
      "buckets": [
        {
          "key_as_string": "2025-03-31T00:00:00.000Z",
          "key": 1743379200000,
          "doc_count": 1048,
          "weekly_sales": {
            "value": 79448.60546875,
            "value_as_string": "$79,448.61"
          }
        },
        {
          "key_as_string": "2025-04-07T00:00:00.000Z",
          "key": 1743984000000,
          "doc_count": 1048,
          "weekly_sales": {
            "value": 78208.4296875,
            "value_as_string": "$78,208.43"
          }
        },
        {
          "key_as_string": "2025-04-14T00:00:00.000Z",
          "key": 1744588800000,
          "doc_count": 1073,
          "weekly_sales": {
            "value": 81277.296875,
            "value_as_string": "$81,277.30"
          }
        }
      ]
    }
  }
}

```

> 由于它返回布尔值而不是数值，因此 buckets_selector 聚合不采用格式参数。在此示例中，格式化的指标由 sum 子聚合在 value_as_string 结果中返回。将此与 bucket_script 聚合中的示例进行对比。

