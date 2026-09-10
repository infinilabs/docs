---
title: "Fail 处理器"
date: 0001-01-01
summary: "Fail 处理器 #  fail 处理器在索引过程中执行数据转换和丰富化非常有用。fail 处理器的主要用例是在满足某些条件时失败索引操作。
以下是为 fail 处理器提供的语法：
&#34;fail&#34;: { &#34;if&#34;: &#34;ctx.foo == &#39;bar&#39;&#34;, &#34;message&#34;: &#34;Custom error message&#34; } 配置参数 #  下表列出了 fail 处理器所需的和可选参数。
   参数 是否必填 描述     message 必填 要包含在失败响应中的自定义错误消息。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# Fail 处理器

`fail` 处理器在索引过程中执行数据转换和丰富化非常有用。`fail` 处理器的主要用例是在满足某些条件时失败索引操作。


以下是为 `fail` 处理器提供的语法：

```
"fail": {
  "if": "ctx.foo == 'bar'",
  "message": "Custom error message"
  }
```

## 配置参数

下表列出了 `fail` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                              |
| ---------------- | -------- | --------------------------------------------------------------------------------- |
| `message`        | 必填     | 要包含在失败响应中的自定义错误消息。                                              |
| `description`    | 可选     | 处理器的一个简要描述。                                                            |
| `if`             | 可选     | 处理器运行的条件。                                                                |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。 |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                  |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                            |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `fail-log-pipeline` 的管道，该管道使用 `fail` 处理器故意使管道执行失败，以处理日志事件：

```
PUT _ingest/pipeline/fail-log-pipeline
{
  "description": "A pipeline to test the fail processor for log events",
  "processors": [
    {
      "fail": {
        "if": "ctx.user_info.contains('password') || ctx.user_info.contains('credit card')",
        "message": "Document containing personally identifiable information (PII) cannot be indexed!"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/fail-log-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "user_info": "Sensitive information including credit card"
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
      "error": {
        "root_cause": [
          {
            "type": "fail_processor_exception",
            "reason": "Document containing personally identifiable information (PII) cannot be indexed!"
          }
        ],
        "type": "fail_processor_exception",
        "reason": "Document containing personally identifiable information (PII) cannot be indexed!"
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=fail-log-pipeline
{
  "user_info": "Sensitive information including credit card"
}
```

请求因字符串 `credit card` 出现在 `user_info` 中而未能将日志事件索引到索引 `testindex1` 。以下响应包括在失败处理器中指定的自定义错误消息：

```
"error": {
    "root_cause": [
      {
        "type": "fail_processor_exception",
        "reason": "Document containing personally identifiable information (PII) cannot be indexed!"
      }
    ],
    "type": "fail_processor_exception",
    "reason": "Document containing personally identifiable information (PII) cannot be indexed!"
  },
  "status": 500
}
```

### 步骤 4（可选）：检索文档

由于日志事件因管道失败而未索引，尝试检索它会导致文档未找到错误 `"found": false` :

```
GET testindex1/_doc/1
```

返回内容

```
{
  "_index": "testindex1",
  "_id": "1",
  "found": false
}
```

