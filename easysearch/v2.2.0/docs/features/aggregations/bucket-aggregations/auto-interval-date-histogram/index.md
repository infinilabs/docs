---
title: "自动间隔日期直方图聚合（Auto Date Histogram）"
date: 0001-01-01
summary: "自动间隔日期直方图聚合 #  与日期直方图聚合类似，其中你必须指定一个间隔，auto_date_histogram 是一个多分组聚合，根据你提供的分组数量和数据的时范围自动创建日期直方图分组。返回的实际分组数量总是小于或等于你指定的分组数量。当你在处理时间序列数据并希望在不同时间间隔上可视化或分析数据，而不需要手动指定间隔大小时，这种聚合特别有用。
相关指南（先读这些） #    聚合基础  聚合场景实践  时间序列建模  间隔参数 #  分组间隔是根据收集的数据选择的，以确保返回的分组数量小于或等于请求的数量。
下表列出了每个时间单位可能的返回间隔。
   单位 间隔参数     Seconds 1、5、10 和 30 的倍数   Minutes 1、5、10 和 30 的倍数   Hours 1、3 和 12 的倍数   Days 1 和 7 的倍数   Months 1 和 3 的倍数   Years 1、5、10、20、50 和 100 的倍数    如果一个聚合返回的分组太多（例如，每天一个分组），Easysearch 会自动减少分组的数量以确保结果可管理。它不会返回请求的确切数量的每日分组，而是会减少大约 1/7。例如，如果你请求 70 个分组，但数据中包含太多的每日间隔，Easysearch 可能只会返回 10 个分组，将数据分组到更大的间隔（如周）中，以避免结果数量过多。这有助于优化聚合，并在数据过多时防止过多细节。"
---


# 自动间隔日期直方图聚合

与日期直方图聚合类似，其中你必须指定一个间隔，`auto_date_histogram` 是一个多分组聚合，根据你提供的分组数量和数据的时范围自动创建日期直方图分组。返回的实际分组数量总是小于或等于你指定的分组数量。当你在处理时间序列数据并希望在不同时间间隔上可视化或分析数据，而不需要手动指定间隔大小时，这种聚合特别有用。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})
- [时间序列建模]({{< relref "/docs/best-practices/data-modeling/time-series.md" >}})

## 间隔参数

分组间隔是根据收集的数据选择的，以确保返回的分组数量小于或等于请求的数量。

下表列出了每个时间单位可能的返回间隔。

| 单位    | 间隔参数                       |
| ------- | ------------------------------ |
| Seconds | 1、5、10 和 30 的倍数          |
| Minutes | 1、5、10 和 30 的倍数          |
| Hours   | 1、3 和 12 的倍数              |
| Days    | 1 和 7 的倍数                  |
| Months  | 1 和 3 的倍数                  |
| Years   | 1、5、10、20、50 和 100 的倍数 |

如果一个聚合返回的分组太多（例如，每天一个分组），Easysearch 会自动减少分组的数量以确保结果可管理。它不会返回请求的确切数量的每日分组，而是会减少大约 1/7。例如，如果你请求 70 个分组，但数据中包含太多的每日间隔，Easysearch 可能只会返回 10 个分组，将数据分组到更大的间隔（如周）中，以避免结果数量过多。这有助于优化聚合，并在数据过多时防止过多细节。

## 参考样例

在以下示例中，你将搜索一个包含博客文章的索引。

首先，为这个索引创建一个映射，并将 `date_posted` 字段指定为 `date` 类型：

```
PUT blogs
{
  "mappings" : {
    "properties" :  {
      "date_posted" : {
        "type" : "date",
        "format" : "yyyy-MM-dd"
      }
    }
  }
}

```

接下来，将以下文档索引到 `blogs` 索引中：

```
PUT blogs/_doc/1
{
  "name": "Semantic search in Easysearch",
  "date_posted": "2022-04-17"
}

PUT blogs/_doc/2
{
  "name": "Sparse search in Easysearch",
  "date_posted": "2022-05-02"
}

PUT blogs/_doc/3
{
  "name": "Distributed tracing with Data Prepper",
  "date_posted": "2022-04-25"
}

PUT blogs/_doc/4
{
  "name": "Observability in Easysearch",
  "date_posted": "2023-03-23"
}
```

要使用 `auto_date_histogram` 聚合，请指定包含日期或时间戳值的字段。例如，要将博客文章按 date_posted 聚合到两个分组中，请发送以下请求：

```
GET /blogs/_search
{
  "size": 0,
  "aggs": {
    "histogram": {
      "auto_date_histogram": {
        "field": "date_posted",
        "buckets": 2
      }
    }
  }
}
```

返回内容显示博客文章被聚合到两个分组中。间隔被自动设置为 1 年，所有三个 2022 年的博客文章被收集到一个分组中，而 2023 年的博客文章在另一个分组中：

```
{
  "took": 20,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 4,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "histogram": {
      "buckets": [
        {
          "key_as_string": "2022-01-01",
          "key": 1640995200000,
          "doc_count": 3
        },
        {
          "key_as_string": "2023-01-01",
          "key": 1672531200000,
          "doc_count": 1
        }
      ],
      "interval": "1y"
    }
  }
}
```

### 分组信息

每个分组包含以下信息：

```
{
  "key_as_string": "2023-01-01",
  "key": 1672531200000,
  "doc_count": 1
}
```

在 Easysearch 中，日期内部存储为 64 位整数，表示自纪元以来的毫秒时间戳。在聚合响应中，每个分组 key 以时间戳的形式返回。 key_as_string 值显示相同的时间戳，但根据 format 参数格式化为日期字符串。 doc_count 字段包含分组中的文档数量。

## 参数说明

自动间隔日期直方图聚合接受以下参数。

| 参数               | 数据类型 | 描述                                                                                                                                                                                                          |
| ------------------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`            | String   | 要对其聚合的字段。该字段必须包含日期或时间戳值。必须提供 `field` 或 `script` 。                                                                                                                               |
| `buckets`          | Integer  | 期望的分组数量。返回的分组数量小于或等于期望的分组数量。可选。默认为 10 。                                                                                                                                    |
| `minimum_interval` | String   | 要使用的最小间隔。指定最小间隔可以使聚合过程更高效。有效值为 `year` 、 `month` 、 `day` 、 `hour` 、 `minute` 和 `second` 。可选。                                                                            |
| `time_zone`        | String   | 指定使用除默认（UTC）之外的时间区进行分组划分和舍入。您可以指定 `time_zone` 参数为 UTC 偏移量，例如 -04:00 ，或 IANA 时间区 ID，例如 `America/New_York` 。可选。默认为 UTC 。有关更多信息，请参阅 Time zone。 |
| `format`           | String   | 返回表示分组键的日期的格式。可选。默认为字段映射中指定的格式。有关更多信息，请参阅 Date format。                                                                                                              |
| `script`           | String   | 一个用于将值聚合到分组中的文档级或值级脚本。必须使用 `field` 或 `script` 。                                                                                                                                   |
| `missing`          | String   | 指定如何处理字段值缺失的文档。默认情况下，此类文档会被忽略。如果你在 `missing` 参数中指定一个日期值，所有字段值缺失的文档都会被收集到指定日期的分组中。                                                       |

## 关于日期格式

如果你没有指定 `format` 参数，将使用字段映射中定义的格式（如前一个响应所示）。要修改格式，请指定 `format` 参数：

```
GET /blogs/_search
{
  "size": 0,
  "aggs": {
    "histogram": {
      "auto_date_histogram": {
        "field": "date_posted",
        "format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}

```

`key_as_string` 字段现在以指定的格式返回：

```
{
  "key_as_string": "2023-01-01 00:00:00",
  "key": 1672531200000,
  "doc_count": 1
}
```

或者，您可以指定一种内置的日期格式：

```
GET /blogs/_search
{
  "size": 0,
  "aggs": {
    "histogram": {
      "auto_date_histogram": {
        "field": "date_posted",
        "format": "basic_date_time_no_millis"
      }
    }
  }
}

```

`key_as_string` 字段现在以指定的格式返回：

```
{
  "key_as_string": "20230101T000000Z",
  "key": 1672531200000,
  "doc_count": 1
}
```

## 时区使用

默认情况下，日期以 UTC 格式存储和处理。 `time_zone` 参数允许您为分分组指定不同的时区。您可以指定 `time_zone` 参数为 UTC 偏移量，例如 -04:00 ，或 IANA 时区 ID，例如 `America/New_York` 。

例如，将以下文档索引到索引中：

```
PUT blogs1/_doc/1
{
  "name": "Semantic search in Easysearch",
  "date_posted": "2022-04-17T01:00:00.000Z"
}

PUT blogs1/_doc/2
{
  "name": "Sparse search in Easysearch",
  "date_posted": "2022-04-17T04:00:00.000Z"
}

```

首先，不指定时区运行聚合操作：

```
GET /blogs1/_search
{
  "size": 0,
  "aggs": {
    "histogram": {
      "auto_date_histogram": {
        "field": "date_posted",
        "buckets": 2,
        "format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}
```

返回内容包含两个 3 小时的分组，从 2022 年 4 月 17 日凌晨 UTC 开始：

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
      "value": 2,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "histogram": {
      "buckets": [
        {
          "key_as_string": "2022-04-17 01:00:00",
          "key": 1650157200000,
          "doc_count": 1
        },
        {
          "key_as_string": "2022-04-17 04:00:00",
          "key": 1650168000000,
          "doc_count": 1
        }
      ],
      "interval": "3h"
    }
  }
}

```

现在，指定一个 `time_zone` 的 -02:00 ：

```
GET /blogs1/_search
{
  "size": 0,
  "aggs": {
    "histogram": {
      "auto_date_histogram": {
        "field": "date_posted",
        "buckets": 2,
        "format": "yyyy-MM-dd HH:mm:ss",
        "time_zone": "-02:00"
      }
    }
  }
}
```

返回内容包含两个分组，其中开始时间偏移 2 小时，从 2022 年 4 月 16 日 23:00 开始：

```
{
  "took": 17,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "histogram": {
      "buckets": [
        {
          "key_as_string": "2022-04-16 23:00:00",
          "key": 1650157200000,
          "doc_count": 1
        },
        {
          "key_as_string": "2022-04-17 02:00:00",
          "key": 1650168000000,
          "doc_count": 1
        }
      ],
      "interval": "3h"
    }
  }
}

```

> 在使用有夏令时（DST）变化的时区时，接近转换点的分组的大小可能与相邻分组的大小略有不同。

