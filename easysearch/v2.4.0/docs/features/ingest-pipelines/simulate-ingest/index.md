---
title: "模拟管道"
date: 0001-01-01
summary: "模拟管道 #  使用 _simulate API 测试管道效果，不会实际索引文档。这是调试管道逻辑的最佳工具。
请求格式 #  模拟已创建的管道：
POST _ingest/pipeline/&lt;pipeline-id&gt;/_simulate 在请求体中内联定义管道进行模拟（无需先创建）：
POST _ingest/pipeline/_simulate 请求内容字段 #  下表列出了用于模拟管道的请求正文字段。
   参数名 是否必需 类型 描述     docs 必需的 数组 用于测试管道的文档内容。   pipeline 可选 对象 要模拟的管道。如果未包含管道标识符，则模拟使用的是最新创建的管道。    docs 字段可以包含下表中列出的子字段。
   参数名 是否必需 类型 描述     source 必需的 对象 文档的 JSON 正文。   id 可选 字符串 一个独特的文档标识符。该标识符不能在索引的其它地方使用。   index 可选 字符串 文档的转换数据出现的索引。    查询参数 #  下表列出了模拟管道的查询参数。"
---


# 模拟管道

使用 `_simulate` API 测试管道效果，不会实际索引文档。这是调试管道逻辑的最佳工具。

## 请求格式

模拟已创建的管道：

```
POST _ingest/pipeline/<pipeline-id>/_simulate
```

在请求体中内联定义管道进行模拟（无需先创建）：

```
POST _ingest/pipeline/_simulate
```

## 请求内容字段

下表列出了用于模拟管道的请求正文字段。

| 参数名     | 是否必需 | 类型 | 描述                                                               |
| ---------- | -------- | ---- | ------------------------------------------------------------------ |
| `docs`     | 必需的   | 数组 | 用于测试管道的文档内容。                                           |
| `pipeline` | 可选     | 对象 | 要模拟的管道。如果未包含管道标识符，则模拟使用的是最新创建的管道。 |

`docs` 字段可以包含下表中列出的子字段。
| 参数名 | 是否必需 | 类型 | 描述 |
| ---------- | -------- | ---- | ------------------------------------------------------------------ |
| `source` | 必需的 | 对象 | 文档的 JSON 正文。 |
| `id` | 可选 | 字符串 | 一个独特的文档标识符。该标识符不能在索引的其它地方使用。 |
| `index` | 可选 | 字符串 | 文档的转换数据出现的索引。 |

## 查询参数

下表列出了模拟管道的查询参数。

| 参数名    | 类型   | 描述                                               |
| --------- | ------ | -------------------------------------------------- |
| `verbose` | 布尔值 | 详细模式。在执行的管道中显示每个处理器的数据输出。 |

### 示例：指定路径中的管道

```
POST /_ingest/pipeline/my-pipeline/_simulate
{
  "docs": [
    {
      "_index": "my-index",
      "_id": "1",
      "_source": {
        "grad_year": 2024,
        "graduated": false,
        "name": "John Doe"
      }
    },
    {
      "_index": "my-index",
      "_id": "2",
      "_source": {
        "grad_year": 2025,
        "graduated": false,
        "name": "Jane Doe"
      }
    }
  ]
}
```

返回内容如下：

```
{
  "docs": [
    {
      "doc": {
        "_index": "my-index",
        "_id": "1",
        "_source": {
          "name": "JOHN DOE",
          "grad_year": 2023,
          "graduated": true
        },
        "_ingest": {
          "timestamp": "2023-06-20T23:19:54.635306588Z"
        }
      }
    },
    {
      "doc": {
        "_index": "my-index",
        "_id": "2",
        "_source": {
          "name": "JANE DOE",
          "grad_year": 2023,
          "graduated": true
        },
        "_ingest": {
          "timestamp": "2023-06-20T23:19:54.635746046Z"
        }
      }
    }
  ]
}
```

### 示例：详细模式

当使用 `verbose` 参数设置为 `true` 运行上一个请求时，响应将显示每个文档的转换序列。例如，对于 ID 为 1 的文档，响应包含按顺序应用管道中每个处理器的结果：

```
{
  "docs": [
    {
      "processor_results": [
        {
          "processor_type": "set",
          "status": "success",
          "description": "Sets the graduation year to 2023",
          "doc": {
            "_index": "my-index",
            "_id": "1",
            "_source": {
              "name": "John Doe",
              "grad_year": 2023,
              "graduated": false
            },
            "_ingest": {
              "pipeline": "my-pipeline",
              "timestamp": "2023-06-20T23:23:26.656564631Z"
            }
          }
        },
        {
          "processor_type": "set",
          "status": "success",
          "description": "Sets 'graduated' to true",
          "doc": {
            "_index": "my-index",
            "_id": "1",
            "_source": {
              "name": "John Doe",
              "grad_year": 2023,
              "graduated": true
            },
            "_ingest": {
              "pipeline": "my-pipeline",
              "timestamp": "2023-06-20T23:23:26.656564631Z"
            }
          }
        },
        {
          "processor_type": "uppercase",
          "status": "success",
          "doc": {
            "_index": "my-index",
            "_id": "1",
            "_source": {
              "name": "JOHN DOE",
              "grad_year": 2023,
              "graduated": true
            },
            "_ingest": {
              "pipeline": "my-pipeline",
              "timestamp": "2023-06-20T23:23:26.656564631Z"
            }
          }
        }
      ]
    }
  ]
}
```

### 示例：在请求体中指定管道

或者，您可以直接在请求体中指定一个管道，而无需先创建一个管道：

```
POST /_ingest/pipeline/_simulate
{
  "pipeline" :
  {
    "description": "Splits text on white space characters",
    "processors": [
      {
        "csv" : {
          "field" : "name",
          "separator": ",",
          "target_fields": ["last_name", "first_name"],
          "trim": true
        }
      },
      {
      "uppercase": {
        "field": "last_name"
      }
    }
    ]
  },
  "docs": [
    {
      "_index": "second-index",
      "_id": "1",
      "_source": {
        "name": "Doe,John"
      }
    },
    {
      "_index": "second-index",
      "_id": "2",
      "_source": {
        "name": "Doe, Jane"
      }
    }
  ]
}
```

返回内容如下：

```
{
  "docs": [
    {
      "doc": {
        "_index": "second-index",
        "_id": "1",
        "_source": {
          "name": "Doe,John",
          "last_name": "DOE",
          "first_name": "John"
        },
        "_ingest": {
          "timestamp": "2023-08-24T19:20:44.816219673Z"
        }
      }
    },
    {
      "doc": {
        "_index": "second-index",
        "_id": "2",
        "_source": {
          "name": "Doe, Jane",
          "last_name": "DOE",
          "first_name": "Jane"
        },
        "_ingest": {
          "timestamp": "2023-08-24T19:20:44.816492381Z"
        }
      }
    }
  ]
}
```

