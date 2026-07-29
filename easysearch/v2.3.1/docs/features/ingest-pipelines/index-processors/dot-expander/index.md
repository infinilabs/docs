---
title: "点展开器"
date: 0001-01-01
summary: "点展开器 #  dot_expander 处理器是一个帮助你处理层次化数据的工具。它将包含点的字段转换为对象字段，使它们能够被管道中的其他处理器访问。如果没有这种转换，包含点的字段将无法被处理。
以下是为 dot_expander 处理器提供的语法：
{ &#34;dot_expander&#34;: { &#34;field&#34;: &#34;field.to.expand&#34; } } 配置参数 #  下表列出了 dot_expander 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 要展开成对象字段的字段。支持模板使用。   path 可选 此字段仅在要展开的字段嵌套在其他对象字段内时才需要。这是因为 field 参数仅识别叶字段。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 点展开器

`dot_expander` 处理器是一个帮助你处理层次化数据的工具。它将包含点的字段转换为对象字段，使它们能够被管道中的其他处理器访问。如果没有这种转换，包含点的字段将无法被处理。


以下是为 `dot_expander` 处理器提供的语法：

```
{
  "dot_expander": {
    "field": "field.to.expand"
  }
}
```

## 配置参数

下表列出了 `dot_expander` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                    |
| ---------------- | -------- | --------------------------------------------------------------------------------------- |
| `field`          | 必填     | 要展开成对象字段的字段。支持模板使用。                                                  |
| `path`           | 可选     | 此字段仅在要展开的字段嵌套在其他对象字段内时才需要。这是因为 `field` 参数仅识别叶字段。 |
| `description`    | 可选     | 处理器的一个简要描述。                                                                  |
| `if`             | 可选     | 处理器运行的条件。                                                                      |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。       |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                        |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                  |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个 `dot_expander` 处理器，该处理器将展开名为 `user.address.city` 和 `user.address.state` 的两个字段到嵌套对象中：

```
PUT /_ingest/pipeline/dot-expander-pipeline
{
  "description": "Dot expander processor",
  "processors": [
    {
      "dot_expander": {
        "field": "user.address.city"
      }
    },
    {
      "dot_expander":{
       "field": "user.address.state"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/dot-expander-pipeline/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "user.address.city": "New York",
        "user.address.state": "NY"
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
          "user": {
            "address": {
              "city": "New York",
              "state": "NY"
            }
          }
        },
        "_ingest": {
          "timestamp": "2024-01-17T01:32:56.501346717Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=dot-expander-pipeline
{
  "user.address.city": "Denver",
  "user.address.state": "CO"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

返回内容中指定的字段已展开为嵌套字段：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 1,
  "_seq_no": 3,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "user": {
      "address": {
        "city": "Denver",
        "state": "CO"
      }
    }
  }
}
```

## path 参数

您可以使用 `path` 参数来指定对象中点分隔字段的路径。例如，以下管道指定了位于 `user` 对象中的 `address.city` 字段：

```
PUT /_ingest/pipeline/dot-expander-pipeline
{
  "description": "Dot expander processor",
  "processors": [
    {
      "dot_expander": {
        "field": "address.city",
        "path": "user"
      }
    },
    {
      "dot_expander":{
       "field": "address.state",
       "path": "user"
      }
    }
  ]
}
```

您可以按照以下方式模拟管道：

```
POST _ingest/pipeline/dot-expander-pipeline/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "user": {
          "address.city": "New York",
          "address.state": "NY"
        }
      }
    }
  ]
}
```

`dot_expander` 处理器将文档转换为以下结构：

```
{
  "user": {
    "address": {
      "city": "New York",
      "state": "NY"
    }
  }
}
```

## 字段名称冲突

如果已存在与 `dot_expander` 处理器应展开的值路径相同的字段，处理器将两个值合并为一个数组。

考虑以下展开字段 `user.name` 的管道：

```
PUT /_ingest/pipeline/dot-expander-pipeline
{
  "description": "Dot expander processor",
  "processors": [
    {
      "dot_expander": {
        "field": "user.name"
      }
    }
  ]
}
```

您可以使用包含两个具有完全相同路径 `user.name` 的值的文档来模拟该管道：

```
POST _ingest/pipeline/dot-expander-pipeline/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "user.name": "John",
        "user": {
          "name": "Steve"
        }
      }
    }
  ]
}
```

返回内容中确认值已合并到数组中：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "user": {
            "name": [
              "Steve",
              "John"
            ]
          }
        },
        "_ingest": {
          "timestamp": "2024-01-17T01:44:57.420220551Z"
        }
      }
    }
  ]
}
```

如果一个字段包含相同的名称但不同的路径，那么该字段需要重命名。例如，以下 `_simulate` 调用会返回一个解析异常：

```
POST _ingest/pipeline/dot-expander-pipeline/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "user": "John",
        "user.name": "Steve"
      }
    }
  ]
}
```

为了避免解析异常，首先使用 `rename` 处理器重命名字段：

```
PUT /_ingest/pipeline/dot-expander-pipeline
{
  "processors" : [
    {
      "rename" : {
        "field" : "user",
        "target_field" : "user.name"
      }
    },
    {
      "dot_expander": {
        "field": "user.name"
      }
    }
  ]
}
```

现在您可以模拟管道：

```
POST _ingest/pipeline/dot-expander-pipeline/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "user": "John",
        "user.name": "Steve"
      }
    }
  ]
}
```

返回内容中确认字段已合并：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "user": {
            "name": [
              "John",
              "Steve"
            ]
          }
        },
        "_ingest": {
          "timestamp": "2024-01-17T01:52:12.864432419Z"
        }
      }
    }
  ]
}
```

