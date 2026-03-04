---
title: "桶脚本聚合（Bucket Script）"
date: 0001-01-01
summary: "桶脚本聚合 #  bucket_script 聚合是一个父管道聚合，它执行脚本以跨一组存储分组执行每个存储分组的数字计算。使用 bucket_script 聚合对分分组聚合中的多个指标执行自定义数值计算。例如，您可以：
 计算派生指标和复合指标。 使用 if/else 语句应用条件逻辑。 计算特定于业务的 KPI，例如自定义评分指标。  相关指南（先读这些） #    聚合基础  聚合场景实践  参数说明 #  bucket_script 聚合采用以下参数。
   参数 必需/可选 数据类型 描述     buckets_path 必需 Object 一个变量名称到分分组指标的映射，用于识别脚本中使用的指标。这些指标必须是数值型。参见脚本变量 。   script 必需 String 或 Object 要执行的脚本。可以是内联脚本、存储脚本或脚本文件。脚本可以访问通过 buckets_path 参数定义的变量名。必须返回一个数值。   gap_policy 可选 String 应用于缺失数据的策略。有效值为 skip 和 insert_zeros 。默认值为 skip 。详见 数据空缺。   format 可选 String 一个 DecimalFormat 格式化字符串。将在聚合的 value_as_string 参数中返回格式化后的输出。    脚本变量 #  buckets_path 参数将脚本变量名称映射到父聚合的指标。然后可以在脚本中使用这些变量。"
---


# 桶脚本聚合

`bucket_script` 聚合是一个父管道聚合，它执行脚本以跨一组存储分组执行每个存储分组的数字计算。使用 `bucket_script` 聚合对分分组聚合中的多个指标执行自定义数值计算。例如，您可以：

- 计算派生指标和复合指标。
- 使用 if/else 语句应用条件逻辑。
- 计算特定于业务的 KPI，例如自定义评分指标。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

`bucket_script` 聚合采用以下参数。

| 参数           | 必需/可选 | 数据类型         | 描述                                                                                                                   |
| -------------- | --------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `buckets_path` | 必需      | Object           | 一个变量名称到分分组指标的映射，用于识别脚本中使用的指标。这些指标必须是数值型。参见脚本变量 。                        |
| `script`       | 必需      | String 或 Object | 要执行的脚本。可以是内联脚本、存储脚本或脚本文件。脚本可以访问通过 `buckets_path` 参数定义的变量名。必须返回一个数值。 |
| `gap_policy`   | 可选      | String           | 应用于缺失数据的策略。有效值为 `skip` 和 `insert_zeros` 。默认值为 `skip` 。详见 数据空缺。                            |
| `format`       | 可选      | String           | 一个 DecimalFormat 格式化字符串。将在聚合的 value_as_string 参数中返回格式化后的输出。                                 |

## 脚本变量

`buckets_path` 参数将脚本变量名称映射到父聚合的指标。然后可以在脚本中使用这些变量。

> 对于 bucket_script 和 bucket_selector 聚合， buckets_path 参数是一个对象而不是字符串，因为它必须引用多个分组指标。有关 buckets_path 字符串版本的描述，请参阅管道聚合页面。

以下 `buckets_path` 将 `sales_sum` 指标映射到 `total_sales` 脚本变量，并将 `item_count` 指标映射到 `item_count` 脚本变量：

```
"buckets_path": {
  "total_sales": "sales_sum",
  "item_count": "item_count"
}
```

映射的变量可以从 `params` 上下文中访问。例如：

- `params.total_sales`
- `params.item_count`

## 参考样例

以下示例创建了一个按月份间隔的一组日期直方图。 `total_sales` 子聚合计算了每个月销售的所有商品的税后总价。 `vendor_count` 聚合计算了每个月的唯一供应商总数。最后， `avg_vendor_spend` 聚合使用内联脚本计算每个月每个供应商的平均消费金额：

```
GET sample_data_ecommerce/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "month"
      },
      "aggs": {
        "total_sales": {
          "sum": {
            "field": "taxful_total_price"
          }
        },
        "vendor_count": {
          "cardinality": {
            "field": "products.manufacturer.keyword"
          }
        },
        "avg_vendor_spend": {
          "bucket_script": {
            "buckets_path": {
              "sales": "total_sales",
              "vendors": "vendor_count"
            },
            "script": "params.sales / params.vendors",
            "format": "$#,###.00"
          }
        }
      }
    }
  }
}

```

## 返回内容

聚合返回格式化的每月平均供应商支出：

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
      "value": 4675,
      "relation": "eq"
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
          "doc_count": 721,
          "vendor_count": {
            "value": 21
          },
          "total_sales": {
            "value": 53468.1484375
          },
          "avg_vendor_spend": {
            "value": 2546.1023065476193,
            "value_as_string": "$2,546.10"
          }
        },
        {
          "key_as_string": "2025-04-01T00:00:00.000Z",
          "key": 1743465600000,
          "doc_count": 3954,
          "vendor_count": {
            "value": 21
          },
          "total_sales": {
            "value": 297415.98046875
          },
          "avg_vendor_spend": {
            "value": 14162.665736607143,
            "value_as_string": "$14,162.67"
          }
        }
      ]
    }
  }
}
```

