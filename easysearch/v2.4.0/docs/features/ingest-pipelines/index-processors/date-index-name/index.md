---
title: "日期索引名称处理器"
date: 0001-01-01
summary: "日期索引名称处理器 #  date_index_name 处理器用于根据文档中的日期或时间戳字段将文档指向正确的基于时间的索引。处理器将 _index 元数据字段设置为日期数学索引名称表达式。然后处理器从正在处理的文档的 field 字段中获取日期或时间戳并将其格式化为日期数学索引名称表达式。然后提取的日期、index_name_prefix 值和 date_rounding 值被组合以创建日期数学索引表达式。例如，如果 field 字段包含值 2023-10-30T12:43:29.000Z，且 index_name_prefix 设置为 week_index-，date_rounding 设置为 w，则日期数学索引名称表达式为 week_index-2023-10-30。您可以使用 date_formats 字段指定日期数学索引表达式中的日期应如何格式化。
以下是为 date_index_name 处理器提供的语法：
{ &#34;date_index_name&#34;: { &#34;field&#34;: &#34;your_date_field or your_timestamp_field&#34;, &#34;date_rounding&#34;: &#34;rounding_value&#34; } } 配置参数 #  下表列出了 date_index_name 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要附加数据的字段名称。支持模板使用。   date_rounding 必填 索引名称中的日期格式为圆整格式。有效值包括 y （年）、 M （月）、 w （周）、 d （日）、 h （小时）、 m （分钟）和 s （秒）。   date_formats 可选 用于解析日期或时间戳字段的日期格式数组。有效选项包括 Java 时间模式或以下格式之一：ISO8601、UNIX、UNIX_MS 或 TAI64N。默认为 yyyy-MM-dd'T'HH:mm:ss."
---


# 日期索引名称处理器

`date_index_name` 处理器用于根据文档中的日期或时间戳字段将文档指向正确的基于时间的索引。处理器将 `_index` 元数据字段设置为日期数学索引名称表达式。然后处理器从正在处理的文档的 `field` 字段中获取日期或时间戳并将其格式化为日期数学索引名称表达式。然后提取的日期、`index_name_prefix` 值和 `date_rounding` 值被组合以创建日期数学索引表达式。例如，如果 `field` 字段包含值 `2023-10-30T12:43:29.000Z`，且 `index_name_prefix` 设置为 `week_index-`，`date_rounding` 设置为 `w`，则日期数学索引名称表达式为 `week_index-2023-10-30`。您可以使用 `date_formats` 字段指定日期数学索引表达式中的日期应如何格式化。

以下是为 `date_index_name` 处理器提供的语法：

```
{
  "date_index_name": {
    "field": "your_date_field or your_timestamp_field",
    "date_rounding": "rounding_value"
  }
}
```

## 配置参数

下表列出了 `date_index_name` 处理器所需的和可选参数。

| 参数                | 是否必填 | 描述                                                                                                                                                        |
| ------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`             | 必填     | 包含要附加数据的字段名称。支持模板使用。                                                                                                                    |
| `date_rounding`     | 必填     | 索引名称中的日期格式为圆整格式。有效值包括 `y` （年）、 `M` （月）、 `w` （周）、 `d` （日）、 `h` （小时）、 `m` （分钟）和 `s` （秒）。                   |
| `date_formats`      | 可选     | 用于解析日期或时间戳字段的日期格式数组。有效选项包括 Java 时间模式或以下格式之一：ISO8601、UNIX、UNIX_MS 或 TAI64N。默认为 `yyyy-MM-dd'T'HH:mm:ss.SSSXX` 。 |
| `index_name_format` | 可选     | 日期格式。默认为 `yyyy-MM-dd` 。支持模板片段。                                                                                                              |
| `index_name_prefix` | 可选     | 在日期之前附加的索引名称前缀。支持模板片段。                                                                                                                |
| `description`       | 可选     | 处理器的一个简要描述。                                                                                                                                      |
| `if`                | 可选     | 处理器运行的条件。                                                                                                                                          |
| `ignore_failure`    | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                                           |
| `locale`            | 可选     | 解析日期的月份名称和星期几时使用的区域设置。默认为 ENGLISH 。支持模板片段。                                                                                 |
| `on_failure`        | 可选     | 在处理器失败时运行的处理器列表。                                                                                                                            |
| `tag`               | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                                                      |
| `timezone`          | 可选     | 使用解析日期时的时间区域。默认为 UTC 。                                                                                                                     |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道。

以下查询创建了一个名为 `date-index-name1` 的管道，该管道使用 `date_index_name` 处理器将日志索引到月度索引中：

```
PUT /_ingest/pipeline/date-index-name1
{
  "description": "Create weekly index pipeline",
  "processors": [
    {
      "date_index_name": {
        "field": "date_field",
        "index_name_prefix": "week_index-",
        "date_rounding": "w",
        "date_formats": ["YYYY-MM-DD"]
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道。

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/date-index-name1/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "date_field": "2023-10-30"
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
        "_index": "<week_index-{2023-10-01||/w{yyyy-MM-dd|UTC}}>",
        "_id": "1",
        "_source": {
          "date_field": "2023-10-30"
        },
        "_ingest": {
          "timestamp": "2023-11-13T18:23:10.408593092Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档。

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=date-index-name1
{
  "date_field": "2023-10-30"
}
```

请求将文档索引到索引 `week_index-2023-10-23` ，并将同一周内所有带有时标的文档也索引到同一个索引，因为管道按周进行四舍五入。

```
{
  "_index": "week_index-2025-09-29",
  "_id": "1",
  "_version": 4,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 3,
  "_primary_term": 1
}
```

### 步骤 4（可选）：检索文档。

要检索文档，请运行以下查询：

```
GET week_index-2025-09-29/_doc/1
```

