---
title: "控制条件"
date: 0001-01-01
summary: "控制条件 #  在摄取管道中，您可以使用可选的 if 参数来控制处理器是否运行。这允许根据传入的文档内容有条件地执行处理器。条件以 Painless 脚本的形式编写，并针对文档上下文（ ctx ）进行评估。
基本执行条件 #  每个处理器都可以包含一个 if 子句。如果条件评估为 true ，则处理器运行；否则，它将被跳过。
示例：删除调试级别日志 #  以下管道将删除任何 log_level 字段等于 debug 的文档：
PUT _ingest/pipeline/drop_debug_logs { &#34;processors&#34;: [ { &#34;drop&#34;: { &#34;if&#34;: &#34;ctx.log_level == &#39;debug&#39;&#34; } } ] } 示例：索引请求 #  POST logs/_doc/1?pipeline=drop_debug_logs { &#34;message&#34;: &#34;User logged in&#34;, &#34;log_level&#34;: &#34;debug&#34; } 此文档被删除，因为条件评估结果为 true
{ &#34;_index&#34;: &#34;logs&#34;, &#34;_id&#34;: &#34;1&#34;, &#34;_version&#34;: -3, &#34;result&#34;: &#34;noop&#34;, &#34;_shards&#34;: { &#34;total&#34;: 0, &#34;successful&#34;: 0, &#34;failed&#34;: 0 } } 字段嵌套时的空值安全检查 #  当处理嵌套字段时，避免空指针异常很重要。在 Painless 脚本中使用空值安全 ?"
---


# 控制条件

在摄取管道中，您可以使用可选的 `if` 参数来控制处理器是否运行。这允许根据传入的文档内容有条件地执行处理器。条件以 Painless 脚本的形式编写，并针对文档上下文（ `ctx` ）进行评估。

## 基本执行条件

每个处理器都可以包含一个 `if` 子句。如果条件评估为 `true` ，则处理器运行；否则，它将被跳过。

### 示例：删除调试级别日志

以下管道将删除任何 `log_level` 字段等于 `debug` 的文档：

```
PUT _ingest/pipeline/drop_debug_logs
{
  "processors": [
    {
      "drop": {
        "if": "ctx.log_level == 'debug'"
      }
    }
  ]
}
```

### 示例：索引请求

```
POST logs/_doc/1?pipeline=drop_debug_logs
{
  "message": "User logged in",
  "log_level": "debug"
}
```

此文档被删除，因为条件评估结果为 `true`

```
{
  "_index": "logs",
  "_id": "1",
  "_version": -3,
  "result": "noop",
  "_shards": {
    "total": 0,
    "successful": 0,
    "failed": 0
  }
}
```

## 字段嵌套时的空值安全检查

当处理嵌套字段时，避免空指针异常很重要。在 Painless 脚本中使用空值安全 `?.` 运算符。

### 示例：根据嵌套字段删除文档

以下删除处理器仅在嵌套 `app.env` 字段存在且等于 `debug` 时执行：

```
PUT _ingest/pipeline/drop_debug_env
{
  "processors": [
    {
      "drop": {
        "if": "ctx.app?.env == 'debug'"
      }
    }
  ]
}
```

如果未配置 null 安全的 `?.` 运算符，索引任何不包含 `app.env` 字段的文档将触发以下空指针异常：

```
{
  "error": "IngestProcessorException[ScriptException[runtime error]; nested: NullPointerException[Cannot invoke \"Object.getClass()\" because \"callArgs[0]\" is null];]",
  "status": 400
}
```

## 处理扁平字段

如果你的文档包含扁平字段，例如， `"app.env": "debug"` ，请使用 `dot_expander` 处理器将其转换为嵌套结构：

```
PUT _ingest/pipeline/drop_debug_env
{
  "processors": [
    {
      "dot_expander": {
        "field": "app.env"
      }
    },
    {
      "drop": {
        "if": "ctx.app?.env == 'debug'"
      }
    }
  ]
}
```

## 安全的方法调用

避免在可能为 null 的值上调用方法。请使用常量或 null 检查代替：

```
{
  "drop": {
    "if": "ctx.app?.env != null && ctx.app.env.contains('debug')"
  }
}
```

## 完整示例：多步骤管道处理

以下摄取管道使用三个处理器：

1. `set` : 如果在 `user` 字段中没有提供值，则将 `user` 字段设置为 `guest` 。
2. `set` : 如果提供了 `status_code` 并且它高于 `400` ，则将 `error` 字段设置为 `true` 。
3. `drop` : 如果 `app.env` 字段等于 `debug` ，则删除整个文档。

```
PUT _ingest/pipeline/logs_processing
{
  "processors": [
    {
      "set": {
        "field": "user",
        "value": "guest",
        "if": "ctx.user == null"
      }
    },
    {
      "set": {
        "field": "error",
        "value": true,
        "if": "ctx.status_code != null && ctx.status_code >= 400"
      }
    },
    {
      "drop": {
        "if": "ctx.app?.env == 'debug'"
      }
    }
  ]
}
```

### 模拟管道

以下模拟请求将条件逻辑应用于三个文档：

```
POST _ingest/pipeline/logs_processing/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "Successful login",
        "status_code": 200
      }
    },
    {
      "_source": {
        "message": "Database error",
        "status_code": 500,
        "user": "alice"
      }
    },
    {
      "_source": {
        "message": "Debug mode trace",
        "app": { "env": "debug" }
      }
    }
  ]
}
```

返回内容展示了处理器如何根据每个条件进行处理：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "status_code": 200,
          "message": "Successful login",
          "user": "guest"
        },
        "_ingest": {
          "timestamp": "2025-04-16T14:04:35.923159885Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "status_code": 500,
          "message": "Database error",
          "error": true,
          "user": "alice"
        },
        "_ingest": {
          "timestamp": "2025-04-16T14:04:35.923198551Z"
        }
      }
    },
    null
  ]
}

```
