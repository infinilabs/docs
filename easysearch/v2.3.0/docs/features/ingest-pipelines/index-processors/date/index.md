---
title: "日期处理器"
date: 0001-01-01
summary: "日期处理器 #  date 处理器用于从文档字段中解析日期，并将解析后的数据添加到新字段中。默认情况下，解析后的数据存储在 @timestamp 字段中。
语法 #  以下是为 date 处理器提供的语法：
{ &#34;date&#34;: { &#34;field&#34;: &#34;date_field&#34;, &#34;formats&#34;: [&#34;yyyy-MM-dd&#39;T&#39;HH:mm:ss.SSSZZ&#34;] } } 配置参数 #  下表列出了 date 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要转换的数据的字段名称。支持模板使用。   formats 必填 期望的日期格式数组。可以是日期格式或以下格式之一：ISO8601、UNIX、UNIX_MS 或 TAI64N。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   locale 可选 解析日期时使用的区域设置。默认为 ENGLISH 。支持模板片段。   on_failure 可选 在处理器失败时运行的处理器列表。   output_format 可选 用于目标字段的日期格式。默认为 yyyy-MM-dd'T'HH:mm:ss."
---


# 日期处理器

`date` 处理器用于从文档字段中解析日期，并将解析后的数据添加到新字段中。默认情况下，解析后的数据存储在 `@timestamp` 字段中。


## 语法

以下是为 `date` 处理器提供的语法：

```
{
  "date": {
    "field": "date_field",
    "formats": ["yyyy-MM-dd'T'HH:mm:ss.SSSZZ"]
  }
}
```

## 配置参数

下表列出了 `date` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                 |
| ---------------- | -------- | ------------------------------------------------------------------------------------ |
| `field`          | 必填     | 包含要转换的数据的字段名称。支持模板使用。                                           |
| `formats`        | 必填     | 期望的日期格式数组。可以是日期格式或以下格式之一：ISO8601、UNIX、UNIX_MS 或 TAI64N。 |
| `description`    | 可选     | 处理器的一个简要描述。                                                               |
| `if`             | 可选     | 处理器运行的条件。                                                                   |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。    |
| `locale`         | 可选     | 解析日期时使用的区域设置。默认为 `ENGLISH` 。支持模板片段。                          |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                     |
| `output_format`  | 可选     | 用于目标字段的日期格式。默认为 `yyyy-MM-dd'T'HH:mm:ss.SSSZZ` 。                      |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                               |
| `target_field`   | 可选     | 存储解析数据的字段名称。默认目标字段为 @timestamp 。                                 |
| `timezone`       | 可选     | 解析日期时使用的时区。默认为 `UTC` 。支持模板片段。                                  |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `date-output-format` 的管道，该管道使用 `date` 处理器将欧洲日期格式转换为美国日期格式，并添加了新的字段 `date_us` ，包含所需的 `output_format` ：

```
PUT /_ingest/pipeline/date-output-format
{
  "description": "Pipeline that converts European date format to US date format",
  "processors": [
    {
      "date": {
        "field" : "date_european",
        "formats" : ["dd/MM/yyyy", "UNIX"],
        "target_field": "date_us",
        "output_format": "MM/dd/yyy",
        "timezone" : "UTC"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/date-output-format/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "date_us": "06/30/2023",
        "date_european": "30/06/2023"
      }
    }
  ]
}
```

返回内容确认管道按预期工作：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "date_us": "06/30/2023",
          "date_european": "30/06/2023"
        },
        "_ingest": {
          "timestamp": "2023-08-22T17:08:46.275195504Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=date-output-format
{
  "date_european": "30/06/2023"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

