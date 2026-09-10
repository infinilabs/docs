---
title: "URL 解码处理器"
date: 0001-01-01
summary: "URL 解码处理器 #  urldecode 处理器在解码日志数据或其他文本字段中的 URL 编码字符串时非常有用。这可以使数据更易于阅读和分析，尤其是在处理包含特殊字符或空格的 URL 或查询参数时。
以下是为 urldecode 处理器提供的语法：
{ &#34;urldecode&#34;: { &#34;field&#34;: &#34;field_to_decode&#34;, &#34;target_field&#34;: &#34;decoded_field&#34; } } 配置参数 #  下表列出了 urldecode 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要解码的 URL 编码字符串的字段。   target_field 可选 存储解码字符串的字段。如果未指定，则解码字符串将存储在原始编码字符串相同的字段中。   ignore_missing 可选 指定处理器是否应忽略不包含指定 field 的文档。如果设置为 true ，则处理器忽略 field 中的缺失值并保持 target_field 不变。默认为 false 。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# URL 解码处理器

`urldecode` 处理器在解码日志数据或其他文本字段中的 URL 编码字符串时非常有用。这可以使数据更易于阅读和分析，尤其是在处理包含特殊字符或空格的 URL 或查询参数时。


以下是为 `urldecode` 处理器提供的语法：

```
{
  "urldecode": {
    "field": "field_to_decode",
    "target_field": "decoded_field"
  }
}
```

## 配置参数

下表列出了 `urldecode` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                                        |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 包含要解码的 URL 编码字符串的字段。                                                                                                         |
| `target_field`   | 可选     | 存储解码字符串的字段。如果未指定，则解码字符串将存储在原始编码字符串相同的字段中。                                                          |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定 `field` 的文档。如果设置为 true ，则处理器忽略 `field` 中的缺失值并保持 `target_field` 不变。默认为 false 。 |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                                      |
| `if`             | 可选     | 处理器运行的条件。                                                                                                                          |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                           |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                                            |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                                      |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `urldecode_pipeline` 的管道，该管道使用 `urldecode` 处理器解码 `encoded_url` 字段中的 URL 编码字符串，并将解码后的字符串存储在 `decoded_url` 字段中：

```
PUT _ingest/pipeline/urldecode_pipeline
{
  "description": "Decode URL-encoded strings",
  "processors": [
    {
      "urldecode": {
        "field": "encoded_url",
        "target_field": "decoded_url"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/urldecode_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "encoded_url": "https://example.com/search?q=hello%20world"
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
          "decoded_url": "https://example.com/search?q=hello world",
          "encoded_url": "https://example.com/search?q=hello%20world"
        },
        "_ingest": {
          "timestamp": "2024-04-25T23:16:44.886165001Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=urldecode_pipeline
{
  "encoded_url": "https://example.com/search?q=url%20decode%20test"
}
```

前一个请求将文档索引到索引 `testindex1` ，并将包含 `encoded_url` 字段的全部文档索引，该字段由 `urldecode_pipeline` 处理以填充 `decoded_url` 字段，如下面的响应所示：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 67,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 68,
  "_primary_term": 47
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

响应包括原始的 `encoded_url` 字段和 `decoded_url` 字段：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 67,
  "_seq_no": 68,
  "_primary_term": 47,
  "found": true,
  "_source": {
    "decoded_url": "https://example.com/search?q=url decode test",
    "encoded_url": "https://example.com/search?q=url%20decode%20test"
  }
}
```

