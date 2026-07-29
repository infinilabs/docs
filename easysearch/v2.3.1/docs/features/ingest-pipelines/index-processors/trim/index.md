---
title: "裁剪处理器"
date: 0001-01-01
summary: "裁剪处理器 #  trim 处理器用于从指定的字段中删除前导和尾随空白字符。
以下是为 trim 处理器提供的语法：
{ &#34;trim&#34;: { &#34;field&#34;: &#34;field_to_trim&#34;, &#34;target_field&#34;: &#34;trimmed_field&#34; } } 配置参数 #  下表列出了 trim 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要修剪文本的字段。   target_field 必填 存储被截断文本的字段。如果未指定，则字段将就地更新。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器忽略字段中的缺失值，并保持 target_field 不变。默认为 false 。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 裁剪处理器

`trim` 处理器用于从指定的字段中删除前导和尾随空白字符。


以下是为 `trim` 处理器提供的语法：

```
{
  "trim": {
    "field": "field_to_trim",
    "target_field": "trimmed_field"
  }
}
```

## 配置参数

下表列出了 `trim` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                                |
| ---------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 包含要修剪文本的字段。                                                                                                              |
| `target_field`   | 必填     | 存储被截断文本的字段。如果未指定，则字段将就地更新。                                                                                |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器忽略字段中的缺失值，并保持 `target_field` 不变。默认为 false 。 |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                              |
| `if`             | 可选     | 处理器运行的条件。                                                                                                                  |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                   |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                                    |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                              |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `trim_pipeline` 的管道，该管道使用 `trim` 处理器从 `raw_text` 字段中删除前导和尾随空白，并将截断后的文本存储在 `trimmed_text` 字段中：

```
PUT _ingest/pipeline/trim_pipeline
{
  "description": "Trim leading and trailing white space",
  "processors": [
    {
      "trim": {
        "field": "raw_text",
        "target_field": "trimmed_text"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/trim_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "raw_text": "   Hello, world!   "
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
          "raw_text": "   Hello, world!   ",
          "trimmed_text": "Hello, world!"
        },
        "_ingest": {
          "timestamp": "2024-04-26T20:58:17.418006805Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=trim_pipeline
{
  "raw_text": "   This is a test document.   "
}
```

请求将文档索引到索引 `testindex1` ，并将所有具有 `raw_text` 字段的文档索引，由 `trim_pipeline` 处理，以填充 `trimmed_text` 字段，如下面的响应所示：

```
"_index": "testindex1",
  "_id": "1",
  "_version": 68,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 70,
  "_primary_term": 47
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

响应包括已去除前后空白的 `trimmed_text` 字段：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 69,
  "_seq_no": 71,
  "_primary_term": 47,
  "found": true,
  "_source": {
    "raw_text": "   This is a test document.   ",
    "trimmed_text": "This is a test document."
  }
}
```

