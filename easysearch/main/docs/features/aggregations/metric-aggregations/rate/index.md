---
title: "速率聚合（Rate）"
date: 0001-01-01
summary: "速率聚合 #  rate 聚合是一个指标聚合，用于计算文档或字段值在指定时间单位内的速率。它必须嵌套在 date_histogram（或 composite 中的 date_histogram 源）内部使用。
rate 聚合特别适合将不同时间粒度的数据统一到相同的速率单位进行对比。例如，当 date_histogram 按月分桶时，你可以用 rate 聚合将每个月的值换算为&quot;每天&quot;或&quot;每年&quot;的速率。
相关指南（先读这些） #    聚合基础  日期直方图聚合  聚合场景实践  参数说明 #     参数 必需/可选 数据类型 描述     field 可选 String 要计算速率的数值字段。如果省略，则计算文档的速率（即文档数/时间单位）。   unit 可选 String 速率的时间单位。有效值：second、minute、hour、day、week、month、quarter、year。如果省略，使用 date_histogram 的 calendar_interval 作为单位。   script 可选 Object 使用脚本动态计算速率值。   missing 可选 Numeric 缺少字段值的文档所使用的替代值。    基本用法：文档速率 #  计算每月的文档数量，并将其换算为每天的速率："
---


# 速率聚合

`rate` 聚合是一个指标聚合，用于计算文档或字段值在指定时间单位内的速率。它**必须嵌套在 `date_histogram`（或 `composite` 中的 `date_histogram` 源）内部使用**。

`rate` 聚合特别适合将不同时间粒度的数据统一到相同的速率单位进行对比。例如，当 `date_histogram` 按月分桶时，你可以用 `rate` 聚合将每个月的值换算为"每天"或"每年"的速率。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [日期直方图聚合]({{< relref "../bucket-aggregations/date-histogram.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

| 参数      | 必需/可选 | 数据类型 | 描述                                                                                                                                                                   |
| --------- | --------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`   | 可选      | String   | 要计算速率的数值字段。如果省略，则计算文档的速率（即文档数/时间单位）。                                                                                                  |
| `unit`    | 可选      | String   | 速率的时间单位。有效值：`second`、`minute`、`hour`、`day`、`week`、`month`、`quarter`、`year`。如果省略，使用 `date_histogram` 的 `calendar_interval` 作为单位。          |
| `script`  | 可选      | Object   | 使用脚本动态计算速率值。                                                                                                                                                |
| `missing` | 可选      | Numeric  | 缺少字段值的文档所使用的替代值。                                                                                                                                        |

## 基本用法：文档速率

计算每月的文档数量，并将其换算为每天的速率：

```
GET logs/_search
{
  "size": 0,
  "aggs": {
    "by_month": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "month"
      },
      "aggs": {
        "daily_rate": {
          "rate": {
            "unit": "day"
          }
        }
      }
    }
  }
}
```

返回内容中，每个月桶内的 `daily_rate` 值是该月文档数量除以该月天数：

```
{
  ...
  "aggregations": {
    "by_month": {
      "buckets": [
        {
          "key_as_string": "2024-01-01T00:00:00.000Z",
          "key": 1704067200000,
          "doc_count": 310,
          "daily_rate": {
            "value": 10.0
          }
        },
        {
          "key_as_string": "2024-02-01T00:00:00.000Z",
          "key": 1706745600000,
          "doc_count": 290,
          "daily_rate": {
            "value": 10.0
          }
        }
      ]
    }
  }
}
```

## 字段值速率

计算某个数值字段在单位时间内的速率。例如，按月统计销售额，并换算为年化速率：

```
GET sales/_search
{
  "size": 0,
  "aggs": {
    "by_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "annual_revenue_rate": {
          "rate": {
            "field": "revenue",
            "unit": "year"
          }
        }
      }
    }
  }
}
```

这将每个月的收入总和乘以 12（近似值，实际会根据月份天数精确计算），得到年化收入速率。

## 不同粒度之间的速率换算

`rate` 聚合会感知日历单位（月份天数、闰年等）。例如：

- `date_histogram` 按 `month` 分桶，`rate` 使用 `unit: day` → 结果 = 月总值 ÷ 该月天数
- `date_histogram` 按 `day` 分桶，`rate` 使用 `unit: month` → 结果 = 日总值 × 该月天数
- `date_histogram` 按 `month` 分桶，`rate` 使用 `unit: year` → 结果 = 月总值 × 12（近似）

## 与 composite 聚合配合

`rate` 也可以在 `composite` 聚合中使用，只要 `composite` 的源之一是 `date_histogram`：

```
GET logs/_search
{
  "size": 0,
  "aggs": {
    "by_host_and_month": {
      "composite": {
        "sources": [
          { "host": { "terms": { "field": "host.keyword" } } },
          { "month": { "date_histogram": { "field": "@timestamp", "calendar_interval": "month" } } }
        ]
      },
      "aggs": {
        "error_rate_per_day": {
          "rate": {
            "unit": "day"
          }
        }
      }
    }
  }
}
```

## 注意事项

1. `rate` 聚合**必须**嵌套在 `date_histogram` 或包含 `date_histogram` 源的 `composite` 聚合内部，否则会报错。
2. `rate` 是叶子聚合（Leaf Aggregation），不能包含子聚合。
3. 如果省略 `field` 和 `script`，`rate` 计算的是文档数量的速率。
4. `unit` 参数支持的有效值必须与 `date_histogram` 的 `calendar_interval` 使用的日历单位一致（均为日历感知单位）。

