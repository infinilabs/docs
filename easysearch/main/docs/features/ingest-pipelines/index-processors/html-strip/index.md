---
title: "HTML 清理处理器"
date: 0001-01-01
summary: "HTML 清理处理器 #  html_strip 处理器从传入文档的字符串字段中移除 HTML 标签。当从网页或其他可能包含 HTML 标记的来源索引数据时，此处理器非常有用。HTML 标签被替换为换行符（\n）。
以下是为 html_strip 处理器提供的语法：
{ &#34;html_strip&#34;: { &#34;field&#34;: &#34;webpage&#34; } } 配置参数 #  下表列出了 html_strip 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 要从中移除 HTML 标签的字符串字段。   target_field 可选 接收去除 HTML 标签后的纯文本版本的字段。如果未指定，则字段将就地更新。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。默认为 false 。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# HTML 清理处理器

`html_strip` 处理器从传入文档的字符串字段中移除 HTML 标签。当从网页或其他可能包含 HTML 标记的来源索引数据时，此处理器非常有用。HTML 标签被替换为换行符（`\n`）。


以下是为 `html_strip` 处理器提供的语法：

```
{
  "html_strip": {
    "field": "webpage"
  }
}
```

## 配置参数

下表列出了 `html_strip` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                              |
| ---------------- | -------- | --------------------------------------------------------------------------------- |
| `field`          | 必填     | 要从中移除 HTML 标签的字符串字段。                                                |
| `target_field`   | 可选     | 接收去除 HTML 标签后的纯文本版本的字段。如果未指定，则字段将就地更新。            |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定字段的文档。默认为 false 。                         |
| `description`    | 可选     | 处理器的一个简要描述。                                                            |
| `if`             | 可选     | 处理器运行的条件。                                                                |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。 |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                  |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                            |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `strip-html-pipeline` 的管道，使用 `html_strip` 处理器从描述字段中删除 HTML 标签，并将处理后的值存储在名为 `cleaned_description` 的新字段中：

```
PUT _ingest/pipeline/strip-html-pipeline
{
  "description": "A pipeline to strip HTML from description field",
  "processors": [
    {
      "html_strip": {
        "field": "description",
        "target_field": "cleaned_description"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/strip-html-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "description": "This is a <b>test</b> description with <i>some</i> HTML tags."
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
          "description": "This is a <b>test</b> description with <i>some</i> HTML tags.",
          "cleaned_description": "This is a test description with some HTML tags."
        },
        "_ingest": {
          "timestamp": "2024-05-22T21:46:11.227974965Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `products` 的索引中：

```
PUT products/_doc/1?pipeline=strip-html-pipeline
{
  "name": "Product 1",
  "description": "This is a <b>test</b> product with <i>some</i> HTML tags."
}
```

响应显示，请求已将文档索引到索引 `products` ，并将索引所有包含 HTML 标签的文档，同时将纯文本版本存储在 `cleaned_description` 字段中：

```
{
  "_index": "products",
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
GET products/_doc/1
```

响应包括原始的 `description` 字段和移除 HTML 标签后的 `cleaned_description` 字段：

```
{
  "_index": "products",
  "_id": "1",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "cleaned_description": "This is a test product with some HTML tags.",
    "name": "Product 1",
    "description": "This is a <b>test</b> product with <i>some</i> HTML tags."
  }
}
```

