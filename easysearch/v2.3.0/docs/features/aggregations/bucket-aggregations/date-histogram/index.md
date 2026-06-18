---
title: "日期直方图聚合（Date Histogram）"
date: 0001-01-01
summary: "日期直方图聚合 #  date_histogram 聚合使用日期计算来为时间序列数据生成直方图。
相关指南（先读这些） #    聚合基础  聚合场景实践  时间序列建模  例如，您可以找到您的网站每月有多少次访问：
GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;logs_per_month&#34;: { &#34;date_histogram&#34;: { &#34;field&#34;: &#34;@timestamp&#34;, &#34;interval&#34;: &#34;month&#34; } } } } 返回内容
... &#34;aggregations&#34; : { &#34;logs_per_month&#34; : { &#34;buckets&#34; : [ { &#34;key_as_string&#34; : &#34;2020-10-01T00:00:00.000Z&#34;, &#34;key&#34; : 1601510400000, &#34;doc_count&#34; : 1635 }, { &#34;key_as_string&#34; : &#34;2020-11-01T00:00:00.000Z&#34;, &#34;key&#34; : 1604188800000, &#34;doc_count&#34; : 6844 }, { &#34;key_as_string&#34; : &#34;2020-12-01T00:00:00."
---


# 日期直方图聚合

`date_histogram` 聚合使用日期计算来为时间序列数据生成直方图。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})
- [时间序列建模]({{< relref "/docs/best-practices/data-modeling/time-series.md" >}})

例如，您可以找到您的网站每月有多少次访问：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "logs_per_month": {
      "date_histogram": {
        "field": "@timestamp",
        "interval": "month"
      }
    }
  }
}

```

返回内容

```
...
"aggregations" : {
  "logs_per_month" : {
    "buckets" : [
      {
        "key_as_string" : "2020-10-01T00:00:00.000Z",
        "key" : 1601510400000,
        "doc_count" : 1635
      },
      {
        "key_as_string" : "2020-11-01T00:00:00.000Z",
        "key" : 1604188800000,
        "doc_count" : 6844
      },
      {
        "key_as_string" : "2020-12-01T00:00:00.000Z",
        "key" : 1606780800000,
        "doc_count" : 5595
      }
    ]
  }
}
```

返回内容包含三个月的日志。如果你绘制这些值，你可以看到你的网站每月请求流量的峰值和低谷。

## 参数说明

`date_histogram` 聚合支持以下参数。

| 参数                | 必需/可选 | 数据类型 | 描述                                                                                                         |
| ------------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------------ |
| `date_histogram	`    | 必填      | Object   | 一个指定日期时间文档字段、间隔、可选格式和时区的对象。                                                       |
| `calendar_interval	` | 必填      | 时间间隔 | 构建每个分组所使用的日期字段。                                                                                 |
| `format	`            | 可选      | String   | 日期格式字符串。如果省略，日期将输出为 64 位自纪元以来的毫秒整数。                                           |
| `time_zone	`         | 可选      | String   | 表示 UTC 时间偏移的字符串，可以是 ISO 8601 UTC 偏移（"-07:00"）或 tz 数据库标识符（"America/Los_Angeles"）。 |

### interval 与 calendar_interval / fixed_interval

`interval` 参数已被标记为弃用，建议使用以下两个参数之一：

| 参数                | 说明                                                                                               |
| ------------------- | -------------------------------------------------------------------------------------------------- |
| `calendar_interval` | 基于日历的间隔，能感知月份天数和闰年。有效值：`minute`、`hour`、`day`、`week`、`month`、`quarter`、`year` |
| `fixed_interval`    | 固定长度间隔，不感知日历。有效值如 `30m`、`1h`、`12h`、`1d`                                         |

例如，按周统计日志量：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "logs_per_week": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "week"
      }
    }
  }
}
```

### 设置时区

日期直方图默认使用 UTC 时区。对于中国地区的数据，通常需要设置时区以获得正确的日期分桶边界：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "logs_per_day": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "day",
        "time_zone": "+08:00",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```

### 空桶填充

默认情况下，没有文档的时间桶不会在结果中出现。设置 `min_doc_count` 为 `0` 可以显示空桶，配合 `extended_bounds` 可以确保结果覆盖指定的完整时间范围：

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "logs_per_day": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "day",
        "min_doc_count": 0,
        "extended_bounds": {
          "min": "2020-10-01",
          "max": "2020-12-31"
        }
      }
    }
  }
}
```

> **提示**：`extended_bounds` 不会过滤文档，只是确保在结果中包含该范围内的所有桶，即使桶内没有文档。

