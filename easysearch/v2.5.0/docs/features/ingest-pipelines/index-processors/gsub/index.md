---
title: "Gsub 处理器"
date: 0001-01-01
summary: "Gsub 处理器 #  gsub 处理器在传入的文档中对字符串字段执行正则表达式搜索和替换操作。如果字段包含字符串数组，则操作应用于数组中的所有元素。然而，如果字段包含非字符串值，处理器将抛出异常。gsub 处理器的用例包括从日志消息或用户生成的内容中删除敏感信息、规范化数据格式或约定（例如，转换日期格式、删除特殊字符），以及从字段值中提取或转换子字符串以进行进一步处理或分析。
以下是为 gsub 处理器提供的语法：
&#34;gsub&#34;: { &#34;field&#34;: &#34;field_name&#34;, &#34;pattern&#34;: &#34;regex_pattern&#34;, &#34;replacement&#34;: &#34;replacement_string&#34; } 配置参数 #  下表列出了 gsub 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 要应用替换的字段。   pattern 必填 要替换的模式。   replacement 必填 将替换匹配模式的字符串。   target_field 可选 要存储解析数据的字段名称。如果未指定 target_field ，则解析数据将替换 field 字段中的原始数据。默认为 field 。   if 可选 处理器运行的条件。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。默认为 false 。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# Gsub 处理器

`gsub` 处理器在传入的文档中对字符串字段执行正则表达式搜索和替换操作。如果字段包含字符串数组，则操作应用于数组中的所有元素。然而，如果字段包含非字符串值，处理器将抛出异常。`gsub` 处理器的用例包括从日志消息或用户生成的内容中删除敏感信息、规范化数据格式或约定（例如，转换日期格式、删除特殊字符），以及从字段值中提取或转换子字符串以进行进一步处理或分析。


以下是为 `gsub` 处理器提供的语法：

```
"gsub": {
  "field": "field_name",
  "pattern": "regex_pattern",
  "replacement": "replacement_string"
}
```

## 配置参数

下表列出了 `gsub` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                               |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------ |
| `field`          | 必填     | 要应用替换的字段。                                                                                                 |
| `pattern`        | 必填     | 要替换的模式。                                                                                                     |
| `replacement`    | 必填     | 将替换匹配模式的字符串。                                                                                           |
| `target_field`   | 可选     | 要存储解析数据的字段名称。如果未指定 `target_field` ，则解析数据将替换 `field` 字段中的原始数据。默认为 `field` 。 |
| `if`             | 可选     | 处理器运行的条件。                                                                                                 |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定字段的文档。默认为 false 。                                                          |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                  |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                   |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                             |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `gsub_pipeline` 的管道，该管道使用 `gsub` 处理器将 `message` 字段中所有出现的单词 `error` 替换为单词 `warning` ：

```
PUT _ingest/pipeline/gsub_pipeline
{
  "description": "Replaces 'error' with 'warning' in the 'message' field",
  "processors": [
    {
      "gsub": {
        "field": "message",
        "pattern": "error",
        "replacement": "warning"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/gsub_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "This is an error message"
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
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "message": "This is an warning message"
        },
        "_ingest": {
          "timestamp": "2024-05-22T19:47:00.645687211Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `logs` 的索引中：

```
PUT logs/_doc/1?pipeline=gsub_pipeline
{
  "message": "This is an error message"
}
```

以下响应显示，请求将文档索引到名为 `logs` 的索引中，并且 `gsub` 处理器将 `message` 字段中所有出现的单词 `error` 替换为单词 `warning` ：

```
{
  "_index": "logs",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET logs/_doc/1
```

以下响应显示了具有修改后的 `message` 字段值的文档：

```
{
  "_index": "logs",
  "_id": "1",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "message": "This is an warning message"
  }
}
```

