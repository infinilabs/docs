---
title: "速率聚合（Rate）"
date: 0001-01-01
summary: "速率聚合 #  rate 聚合是一个指标聚合，用于计算文档或字段值在指定时间单位内的速率。它必须位于 date_histogram 内， 或者作为恰好包含一个 date_histogram 源的 composite 聚合的直接子聚合。
rate 聚合特别适合将不同时间粒度的数据统一到相同的速率单位进行对比。例如，当 date_histogram 按月分桶时，你可以用 rate 聚合将每个月的值换算为&quot;每天&quot;或&quot;每年&quot;的速率。
相关指南（先读这些） #    聚合基础  日期直方图聚合  聚合场景实践  参数说明 #     参数 必需/可选 数据类型 描述     field 可选 String 要计算速率的数值或布尔字段。如果同时省略 field 和 script，每个物理文档计数 1。   unit 可选 String 速率的时间单位。有效值：second、minute、hour、day、week、month、quarter、year。省略时不做时间单位换算。   mode 可选 String 字段值统计方式：sum 或 value_count，默认为 sum。只有设置了 field 或 script 时才能显式指定。   script 可选 Object 使用脚本动态计算速率值。   missing 可选 Numeric 缺少字段值的文档所使用的替代值。   format 可选 String 数值输出格式；设置后响应会包含 value_as_string。    基本用法：文档速率 #  计算每月的文档数量，并将其换算为每天的速率："
---


# 速率聚合

`rate` 聚合是一个指标聚合，用于计算文档或字段值在指定时间单位内的速率。它必须位于 `date_histogram` 内，
或者作为恰好包含一个 `date_histogram` 源的 `composite` 聚合的直接子聚合。

`rate` 聚合特别适合将不同时间粒度的数据统一到相同的速率单位进行对比。例如，当 `date_histogram` 按月分桶时，你可以用 `rate` 聚合将每个月的值换算为"每天"或"每年"的速率。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [日期直方图聚合]({{< relref "../bucket-aggregations/date-histogram.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

## 参数说明

| 参数      | 必需/可选 | 数据类型 | 描述 |
| --------- | --------- | -------- | ---- |
| `field`   | 可选      | String   | 要计算速率的数值或布尔字段。如果同时省略 `field` 和 `script`，每个物理文档计数 1。 |
| `unit`    | 可选      | String   | 速率的时间单位。有效值：`second`、`minute`、`hour`、`day`、`week`、`month`、`quarter`、`year`。省略时不做时间单位换算。 |
| `mode`    | 可选      | String   | 字段值统计方式：`sum` 或 `value_count`，默认为 `sum`。只有设置了 `field` 或 `script` 时才能显式指定。 |
| `script`  | 可选      | Object   | 使用脚本动态计算速率值。 |
| `missing` | 可选      | Numeric  | 缺少字段值的文档所使用的替代值。 |
| `format`  | 可选      | String   | 数值输出格式；设置后响应会包含 `value_as_string`。 |

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

这将每个月的收入总和按月与年的比例换算，得到年化收入速率。

## 字段值计数速率

默认的 `mode: sum` 会累加桶内的全部字段值。使用 `mode: value_count` 时，每个字段值计数 1；多值字段中的
每个值都会单独计数：

```
GET events/_search
{
  "size": 0,
  "aggs": {
    "by_day": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "day"
      },
      "aggs": {
        "values_per_hour": {
          "rate": {
            "field": "measurements",
            "mode": "value_count",
            "unit": "hour"
          }
        }
      }
    }
  }
}
```

## 不同粒度之间的速率换算

`rate` 聚合会感知日历单位（月份天数、闰年等）。例如：

- `date_histogram` 按 `month` 分桶，`rate` 使用 `unit: day` → 结果 = 月总值 ÷ 该月天数
- `date_histogram` 按 `month` 分桶，`rate` 使用 `unit: year` → 结果 = 月总值 × 12
- `date_histogram` 按 `day` 分桶，`rate` 使用 `unit: hour` → 结果 = 日总值 ÷ 24

基于天、小时、分钟或秒的分桶不能换算为 `month`、`quarter` 或 `year`；此类请求会被拒绝。

## 与 composite 聚合配合

`rate` 也可以作为 `composite` 的直接子聚合使用。`composite` 必须恰好包含一个 `date_histogram` 源，
该源在 `sources` 中的位置不受限制：

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

当返回结果包含 `after_key` 时，将它作为下一次请求的 `composite.after` 即可继续翻页；每一页的 rate 都按该页
桶对应的日期宽度计算：

```
GET logs/_search
{
  "size": 0,
  "aggs": {
    "by_host_and_month": {
      "composite": {
        "size": 100,
        "after": {
          "host": "api-01",
          "month": 1706745600000
        },
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

1. `rate` 必须嵌套在 `date_histogram` 内，或作为恰好包含一个 `date_histogram` 源的 `composite` 的直接子聚合，否则会报错。
2. `rate` 是叶子聚合（Leaf Aggregation），不能包含子聚合。
3. 如果省略 `field` 和 `script`，`rate` 按每个物理文档计数 1，不采用 `_doc_count` 权重；显式设置 `mode` 需要同时提供 `field` 或 `script`。
4. 当前日期舍入语义把日历日换算为小时数时固定按 24 小时处理。DST 向前切换日内总值为 23 时结果是 `23/24`，向后切换日内总值为 25 时结果是 `25/24`。

