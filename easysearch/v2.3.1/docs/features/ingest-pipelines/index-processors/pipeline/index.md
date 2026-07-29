---
title: "管道处理器"
date: 0001-01-01
summary: "管道处理器 #  pipeline 处理器允许管道引用和包含另一个预定义的管道。当您有一组需要在多个管道之间共享的常见处理器时，这非常有用。您不必在每个管道中重新定义这些常见处理器，而是可以创建一个包含共享处理器的单独基础管道，然后使用管道处理器从其他管道中引用该基础管道。
以下是为 pipeline 处理器提供的语法：
{ &#34;pipeline&#34;: { &#34;name&#34;: &#34;general-pipeline&#34; } } 配置参数 #  下表列出了 pipeline 处理器所需的和可选参数。
   参数 是否必填 描述     name 必填 要执行的管道的名称。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  步骤 1：创建一个管道 #  以下查询创建了一个名为 general-pipeline 的一般管道，然后创建了一个名为 outer-pipeline 的新管道，该管道引用了 general-pipeline :"
---


# 管道处理器

`pipeline` 处理器允许管道引用和包含另一个预定义的管道。当您有一组需要在多个管道之间共享的常见处理器时，这非常有用。您不必在每个管道中重新定义这些常见处理器，而是可以创建一个包含共享处理器的单独基础管道，然后使用管道处理器从其他管道中引用该基础管道。


以下是为 `pipeline` 处理器提供的语法：

```
{
  "pipeline": {
    "name": "general-pipeline"
  }
}
```

## 配置参数

下表列出了 `pipeline` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                              |
| ---------------- | -------- | --------------------------------------------------------------------------------- |
| `name`           | 必填     | 要执行的管道的名称。                                                              |
| `description`    | 可选     | 处理器的一个简要描述。                                                            |
| `if`             | 可选     | 处理器运行的条件。                                                                |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。 |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                  |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                            |

## 如何使用

### 步骤 1：创建一个管道

以下查询创建了一个名为 `general-pipeline` 的一般管道，然后创建了一个名为 `outer-pipeline` 的新管道，该管道引用了 `general-pipeline` :

```
PUT _ingest/pipeline/general-pipeline
{
  "description": "a general pipeline",
  "processors": [
    {
      "uppercase": {
        "field": "protocol"
      },
      "remove": {
        "field": "name"
      }
    }
  ]
}

PUT _ingest/pipeline/outer-pipeline
{
  "description": "an outer pipeline referencing the general pipeline",
  "processors": [
    {
      "pipeline": {
        "name": "general-pipeline"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/outer-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "protocol": "https",
        "name":"test"
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
          "protocol": "HTTPS"
        },
        "_ingest": {
          "timestamp": "2024-05-24T02:43:43.700735801Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
POST testindex1/_doc/1?pipeline=outer-pipeline
{
  "protocol": "https",
  "name": "test"
}
```

请求将文档中的 `protocol` 字段转换为大写并从索引中移除字段名，如下面的响应所示：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 2,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

响应显示将 `protocol` 字段转换为大写并移除了字段名称的文档：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 2,
  "_seq_no": 1,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "protocol": "HTTPS"
  }
}
```

