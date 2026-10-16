---
title: "分割处理器"
date: 0001-01-01
summary: "分割处理器 #  split 处理器用于根据指定的分隔符将字符串字段拆分为一个子字符串数组。
以下是为 split 处理器提供的语法：
{ &#34;split&#34;: { &#34;field&#34;: &#34;field_to_split&#34;, &#34;separator&#34;: &#34;&lt;delimiter&gt;&#34;, &#34;target_field&#34;: &#34;split_field&#34; } } 配置参数 #  下表列出了 split 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要拆分的字符串的字段。   separator 必填 字符串分割时使用的分隔符。这可以是一个正则表达式模式。   preserve_trailing 可选 如果设置为 true ，则保留结果数组中的空尾字段（例如， '' ）。如果设置为 false ，则从结果数组中删除空尾字段。默认为 false 。   target_field 可选 存储子字符串数组的字段。如果没有指定，则字段将就地更新。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器忽略字段中的缺失值，并保持 target_field 不变。默认为 false 。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 分割处理器

`split` 处理器用于根据指定的分隔符将字符串字段拆分为一个子字符串数组。


以下是为 `split` 处理器提供的语法：

```
{
  "split": {
    "field": "field_to_split",
    "separator": "<delimiter>",
    "target_field": "split_field"
  }
}
```

## 配置参数

下表列出了 `split` 处理器所需的和可选参数。

| 参数                | 是否必填 | 描述                                                                                                                                |
| ------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `field`             | 必填     | 包含要拆分的字符串的字段。                                                                                                          |
| `separator`         | 必填     | 字符串分割时使用的分隔符。这可以是一个正则表达式模式。                                                                              |
| `preserve_trailing` | 可选     | 如果设置为 true ，则保留结果数组中的空尾字段（例如， `''` ）。如果设置为 false ，则从结果数组中删除空尾字段。默认为 false 。        |
| `target_field`      | 可选     | 存储子字符串数组的字段。如果没有指定，则字段将就地更新。                                                                            |
| `ignore_missing`    | 可选     | 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器忽略字段中的缺失值，并保持 `target_field` 不变。默认为 false 。 |
| `description`       | 可选     | 处理器的一个简要描述。                                                                                                              |
| `if`                | 可选     | 处理器运行的条件。                                                                                                                  |
| `ignore_failure`    | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                   |
| `on_failure`        | 可选     | 在处理器失败时运行的处理器列表。                                                                                                    |
| `tag`               | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                              |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `split_pipeline` 的管道，该管道使用 `split` 处理器在 `log_message` 字段上按逗号字符分割，并将结果数组存储在 `log_parts` 字段中：

```
PUT _ingest/pipeline/split_pipeline
{
  "description": "Split log messages by comma",
  "processors": [
    {
      "split": {
        "field": "log_message",
        "separator": ",",
        "target_field": "log_parts"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/split_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "log_message": "error,warning,info"
      }
    }
  ]
}
```

以下示例响应确认管道按预期工作：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "log_message": "error,warning,info",
          "log_parts": [
            "error",
            "warning",
            "info"
          ]
        },
        "_ingest": {
          "timestamp": "2024-04-26T22:29:23.207849376Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=split_pipeline
{
  "log_message": "error,warning,info"
}
```

请求将文档索引到索引 `testindex1` ，并在索引前将 `log_message` 字段以逗号分隔符拆分，如下面的响应所示：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 70,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 72,
  "_primary_term": 47
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

响应显示 `log_message` 字段为以逗号分隔符分割的值数组：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 70,
  "_seq_no": 72,
  "_primary_term": 47,
  "found": true,
  "_source": {
    "log_message": "error,warning,info",
    "log_parts": [
      "error",
      "warning",
      "info"
    ]
  }
}
```

