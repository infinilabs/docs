---
title: "控制条件和处理器"
date: 0001-01-01
summary: "控制条件和处理器 #  在摄取管道中， pipeline 处理器允许根据文档内容条件性地执行不同的子管道。当不同类型的文档需要单独的处理逻辑时，这提供了强大的灵活性。您可以使用 if 参数在 pipeline 处理器中根据字段值、数据类型或内容结构将文档路由到不同的管道。然后，每个管道可以独立应用自己的处理器集。这种方法通过仅在相关位置应用逻辑，保持了管道的模块化和可维护性。
示例：按服务路由日志 #  以下示例演示了如何根据文档中的 service.name 字段将日志路由到不同的子管道。
创建第一个名为 webapp_logs 的管道：
PUT _ingest/pipeline/webapp_logs { &#34;processors&#34;: [ { &#34;set&#34;: { &#34;field&#34;: &#34;log_type&#34;, &#34;value&#34;: &#34;webapp&#34; } } ] } 创建第二个名为 api_logs 的管道：
PUT _ingest/pipeline/api_logs { &#34;processors&#34;: [ { &#34;set&#34;: { &#34;field&#34;: &#34;log_type&#34;, &#34;value&#34;: &#34;api&#34; } } ] } 创建主路由管道名为 service_router ，根据 service.name 将文档路由到相应的管道：
PUT _ingest/pipeline/service_router { &#34;processors&#34;: [ { &#34;pipeline&#34;: { &#34;name&#34;: &#34;webapp_logs&#34;, &#34;if&#34;: &#34;ctx."
---


# 控制条件和处理器

在摄取管道中， `pipeline` 处理器允许根据文档内容条件性地执行不同的子管道。当不同类型的文档需要单独的处理逻辑时，这提供了强大的灵活性。您可以使用 `if` 参数在 `pipeline` 处理器中根据字段值、数据类型或内容结构将文档路由到不同的管道。然后，每个管道可以独立应用自己的处理器集。这种方法通过仅在相关位置应用逻辑，保持了管道的模块化和可维护性。

## 示例：按服务路由日志

以下示例演示了如何根据文档中的 `service.name` 字段将日志路由到不同的子管道。

创建第一个名为 `webapp_logs` 的管道：

```
PUT _ingest/pipeline/webapp_logs
{
  "processors": [
    { "set": { "field": "log_type", "value": "webapp" } }
  ]
}
```

创建第二个名为 `api_logs` 的管道：

```
PUT _ingest/pipeline/api_logs
{
  "processors": [
    { "set": { "field": "log_type", "value": "api" } }
  ]
}
```

创建主路由管道名为 `service_router` ，根据 `service.name` 将文档路由到相应的管道：

```
PUT _ingest/pipeline/service_router
{
  "processors": [
    {
      "pipeline": {
        "name": "webapp_logs",
        "if": "ctx.service?.name == 'webapp'"
      }
    },
    {
      "pipeline": {
        "name": "api_logs",
        "if": "ctx.service?.name == 'api'"
      }
    }
  ]
}
```

使用以下请求来模拟管道：

```
POST _ingest/pipeline/service_router/_simulate
{
  "docs": [
    { "_source": { "service": { "name": "webapp" }, "message": "Homepage loaded" } },
    { "_source": { "service": { "name": "api" }, "message": "GET /v1/users" } },
    { "_source": { "service": { "name": "worker" }, "message": "Task started" } }
  ]
}
```

返回内容中，第一份文档由 `webapp_logs` 管道处理，第二份文档由 `api_logs` 管道处理。第三份文档保持不变，因为它不匹配任何条件：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "log_type": "webapp",
          "message": "Homepage loaded",
          "service": {
            "name": "webapp"
          }
        },
        "_ingest": {
          "timestamp": "2025-04-24T10:54:12.555447087Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "log_type": "api",
          "message": "GET /v1/users",
          "service": {
            "name": "api"
          }
        },
        "_ingest": {
          "timestamp": "2025-04-24T10:54:12.55548442Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "message": "Task started",
          "service": {
            "name": "worker"
          }
        },
        "_ingest": {
          "timestamp": "2025-04-24T10:54:12.555490754Z"
        }
      }
    }
  ]
}

```

## 示例：数据类型处理

您还可以使用管道处理器应用特定类型的管道。以下管道将日志根据 `code` 字段是否为数字，分别发送到 `numeric_handler` 和 `string_handler` 。如果它是类型 `String` ，则发送到 `string_handler` 。

创建第一个名为 `numeric_handler` 的管道：

```
PUT _ingest/pipeline/numeric_handler
{
  "processors": [
    { "set": { "field": "code_type", "value": "numeric" } }
  ]
}
```

创建第二个名为 `string_handler` 的管道：

```
PUT _ingest/pipeline/string_handler
{
  "processors": [
    { "set": { "field": "code_type", "value": "string" } }
  ]
}
```

创建名为 `type_router` 的主路由管道，根据 `code` 字段将文档路由到相应的管道：

```
PUT _ingest/pipeline/type_router
{
  "processors": [
    {
      "pipeline": {
        "name": "numeric_handler",
        "if": "ctx.code instanceof Integer || ctx.code instanceof Long || ctx.code instanceof Double"
      }
    },
    {
      "pipeline": {
        "name": "string_handler",
        "if": "ctx.code instanceof String"
      }
    }
  ]
}
```

使用以下请求来模拟管道：

```
POST _ingest/pipeline/type_router/_simulate
{
  "docs": [
    { "_source": { "code": 404 } },
    { "_source": { "code": "ERR_NOT_FOUND" } }
  ]
}
```

返回的文档由各个子管道添加了新字段 `code_type`

```
{
  "docs": [
    {
      "doc": {
        "_source": {
          "code": 404,
          "code_type": "numeric"
        }
      }
    },
    {
      "doc": {
        "_source": {
          "code": "ERR_NOT_FOUND",
          "code_type": "string"
        }
      }
    }
  ]
}
```

