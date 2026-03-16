---
title: "Drop 处理器"
date: 0001-01-01
summary: "Drop 处理器 #  drop 处理器用于丢弃不进行索引的文档。这可以用于根据某些条件防止文档被索引。例如，您可能使用 drop 处理器来防止缺少重要字段或包含敏感信息的文档被索引。
当 drop 处理器丢弃文档时，它不会引发任何错误，这使得它对于防止索引问题而不会在您的 Easysearch 日志中充斥错误消息非常有用。
语法 #  以下是为 drop 处理器提供的语法：
{ &#34;drop&#34;: { &#34;if&#34;: &#34;ctx.foo == &#39;bar&#39;&#34; } } 配置参数 #  下表列出了 drop 处理器所需的和可选参数。
   参数 是否必填 描述     description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# Drop 处理器

`drop` 处理器用于丢弃不进行索引的文档。这可以用于根据某些条件防止文档被索引。例如，您可能使用 `drop` 处理器来防止缺少重要字段或包含敏感信息的文档被索引。

当 `drop` 处理器丢弃文档时，它不会引发任何错误，这使得它对于防止索引问题而不会在您的 Easysearch 日志中充斥错误消息非常有用。


## 语法

以下是为 `drop` 处理器提供的语法：

```
{
  "drop": {
    "if": "ctx.foo == 'bar'"
  }
}
```

## 配置参数

下表列出了 `drop` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                              |
| ---------------- | -------- | --------------------------------------------------------------------------------- |
| `description`    | 可选     | 处理器的一个简要描述。                                                            |
| `if`             | 可选     | 处理器运行的条件。                                                                |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。 |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                  |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                            |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `drop-pii` 的管道，该管道使用 `drop` 处理器防止包含个人身份信息（PII）的文档被索引：

```
PUT /_ingest/pipeline/drop-pii
{
  "description": "Pipeline that prevents PII from being indexed",
  "processors": [
    {
      "drop": {
        "if" : "ctx.user_info.contains('password') || ctx.user_info.contains('credit card')"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/drop-pii/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "user_info": "Sensitive information including credit card"
      }
    }
  ]
}
```

以下示例响应确认管道按预期工作（文档已被删除）：

```
{
  "docs": [
    null
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中，可以看到 ID 为 1 的文档未进行索引：：

```
PUT testindex1/_doc/1?pipeline=drop-pii
{
  "user_info": "Sensitive information including credit card"
}
```

