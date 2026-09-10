---
title: "复杂逻辑"
date: 0001-01-01
summary: "复杂逻辑 #  在摄取管道中，处理器中的 if 参数可以使用 Painless 脚本来评估复杂条件。这些条件有助于微调文档处理，允许进行高级逻辑，例如类型检查、正则表达式和组合多个标准。
多条件检查 #  您可以将逻辑运算符如 &amp;&amp; （与）、 || （或）和 ! （非）组合起来构建更复杂的条件。以下管道标签将文档标记为 spam ，如果它们包含一个高于 1000 的 error_code ，则将其丢弃：
PUT _ingest/pipeline/spammy_error_handler { &#34;processors&#34;: [ { &#34;set&#34;: { &#34;field&#34;: &#34;tags&#34;, &#34;value&#34;: [&#34;spam&#34;], &#34;if&#34;: &#34;ctx.message != null &amp;&amp; ctx.message.contains(&#39;OutOfMemoryError&#39;)&#34; } }, { &#34;drop&#34;: { &#34;if&#34;: &#34;ctx.tags != null &amp;&amp; ctx.tags.contains(&#39;spam&#39;) &amp;&amp; ctx.error_code != null &amp;&amp; ctx.error_code &gt; 1000&#34; } } ] } 您可以使用以下 _simulate 请求测试管道：
POST _ingest/pipeline/spammy_error_handler/_simulate { &#34;docs&#34;: [ { &#34;_source&#34;: { &#34;message&#34;: &#34;OutOfMemoryError occurred&#34;, &#34;error_code&#34;: 1200 } }, { &#34;_source&#34;: { &#34;message&#34;: &#34;OutOfMemoryError occurred&#34;, &#34;error_code&#34;: 800 } }, { &#34;_source&#34;: { &#34;message&#34;: &#34;All good&#34;, &#34;error_code&#34;: 200 } } ] } 第一份文档被丢弃，因为它包含一个 OutOfMemoryError 字符串和一个高于 1000 的 error_code ："
---


# 复杂逻辑

在摄取管道中，处理器中的 `if` 参数可以使用 Painless 脚本来评估复杂条件。这些条件有助于微调文档处理，允许进行高级逻辑，例如类型检查、正则表达式和组合多个标准。

## 多条件检查

您可以将逻辑运算符如 `&&` （与）、 `||` （或）和 `!` （非）组合起来构建更复杂的条件。以下管道标签将文档标记为 `spam` ，如果它们包含一个高于 `1000` 的 `error_code` ，则将其丢弃：

```
PUT _ingest/pipeline/spammy_error_handler
{
  "processors": [
    {
      "set": {
        "field": "tags",
        "value": ["spam"],
        "if": "ctx.message != null && ctx.message.contains('OutOfMemoryError')"
      }
    },
    {
      "drop": {
        "if": "ctx.tags != null && ctx.tags.contains('spam') && ctx.error_code != null && ctx.error_code > 1000"
      }
    }
  ]
}
```

您可以使用以下 `_simulate` 请求测试管道：

```
POST _ingest/pipeline/spammy_error_handler/_simulate
{
  "docs": [
    { "_source": { "message": "OutOfMemoryError occurred", "error_code": 1200 } },
    { "_source": { "message": "OutOfMemoryError occurred", "error_code": 800 } },
    { "_source": { "message": "All good", "error_code": 200 } }
  ]
}
```

第一份文档被丢弃，因为它包含一个 `OutOfMemoryError` 字符串和一个高于 `1000` 的 `error_code` ：

```
{
  "docs": [
    null,
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "error_code": 800,
          "message": "OutOfMemoryError occurred",
          "tags": [
            "spam"
          ]
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:20:10.704359884Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "error_code": 200,
          "message": "All good"
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:20:10.704369801Z"
        }
      }
    }
  ]
}
```

## 类型安全的评估

使用 `instanceof` 确保在执行操作之前正在使用正确的数据类型。以下管道配置为仅在 `message` 为类型 `String` 且长度超过 `10` 个字符时，添加设置为 `true` 的 `processed` 字段：

```
PUT _ingest/pipeline/string_message_check
{
  "processors": [
    {
      "set": {
        "field": "processed",
        "value": true,
        "if": "ctx.message != null && ctx.message instanceof String && ctx.message.length() > 10"
      }
    }
  ]
}
```

使用以下 `_simulate` 请求测试管道：

```
POST _ingest/pipeline/string_message_check/_simulate
{
  "docs": [
    { "_source": { "message": "short" } },
    { "_source": { "message": "This is a longer message" } },
    { "_source": { "message": 1234567890 } }
  ]
}
```

只有第二份文档添加了新字段：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "message": "short"
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:28:14.040115261Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "processed": true,
          "message": "This is a longer message"
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:28:14.040141469Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "message": 1234567890
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:28:14.040144844Z"
        }
      }
    }
  ]
}

```

## 使用正则表达式

Painless 脚本支持 `=~` 运算符来评估正则表达式。以下管道标志以 `192.168.` 开头的可疑 IP 模式：

```
PUT _ingest/pipeline/flag_suspicious_ips
{
  "processors": [
    {
      "set": {
        "field": "alert",
        "value": "suspicious_ip",
        "if": """ctx.ip != null && ctx.ip =~ /^192\.168\.\d+\.\d+$/"""
      }
    }
  ]
}
```

使用以下 `_simulate` 请求测试管道：

```
POST _ingest/pipeline/flag_suspicious_ips/_simulate
{
  "docs": [
    { "_source": { "ip": "192.168.0.1" } },
    { "_source": { "ip": "10.0.0.1" } }
  ]
}
```

第一份文档添加了 `alert` 字段：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "alert": "suspicious_ip",
          "ip": "192.168.0.1"
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:32:45.367916428Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "ip": "10.0.0.1"
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:32:45.36793772Z"
        }
      }
    }
  ]
}
```

## 组合字段和空值检查

以下管道添加一个 `priority` 字段，如果 `level` 是 `critical` 且提供了 `timestamp` ，则将其设置为 `high` 。该脚本还确保在继续之前所有字段都存在并满足特定条件：

```
PUT _ingest/pipeline/critical_log_handler
{
  "processors": [
    {
      "set": {
        "field": "priority",
        "value": "high",
        "if": "ctx.level != null && ctx.level == 'critical' && ctx.timestamp != null"
      }
    }
  ]
}
```

使用以下 `_simulate` 请求测试管道：

```
POST _ingest/pipeline/critical_log_handler/_simulate
{
  "docs": [
    { "_source": { "level": "critical", "timestamp": "2025-04-01T00:00:00Z" } },
    { "_source": { "level": "info", "timestamp": "2025-04-01T00:00:00Z" } },
    { "_source": { "level": "critical" } }
  ]
}
```

只有第一份文档添加了 `priority` 字段：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "priority": "high",
          "level": "critical",
          "timestamp": "2025-04-01T00:00:00Z"
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:39:25.46840371Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "level": "info",
          "timestamp": "2025-04-01T00:00:00Z"
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:39:25.46843021Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "level": "critical"
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:39:25.468434835Z"
        }
      }
    }
  ]
}

```

## 多条件处理

以下条件：

- 如果该字段尚未存在，则添加一个设置为 `production` 的 `env` 字段。
- 如果 `status` 字段中的值大于或等于 `500` ，则添加一个设置为 `major` 的 `severity` 字段。
- 如果 `env` 字段设置为 `test` 且 `message` 字段包含 `debug` ，则删除该文档。

```
PUT _ingest/pipeline/advanced_log_pipeline
{
  "processors": [
    {
      "set": {
        "field": "env",
        "value": "production",
        "if": "!ctx.containsKey('env')"
      }
    },
    {
      "set": {
        "field": "severity",
        "value": "major",
        "if": "ctx.status != null && ctx.status >= 500"
      }
    },
    {
      "drop": {
        "if": "ctx.env == 'test' && ctx.message?.contains('debug')"
      }
    }
  ]
}
```

使用以下 `_simulate` 请求来测试管道：

```
POST _ingest/pipeline/advanced_log_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "status": 503,
        "message": "Server unavailable"
      }
    },
    {
      "_source": {
        "env": "test",
        "message": "debug log output"
      }
    },
    {
      "_source": {
        "status": 200,
        "message": "OK"
      }
    }
  ]
}
```

在返回内容中，请注意第一个文档添加了 `env: production` 和 `severity: major` 字段。第二个文档被丢弃。第三个文档添加了一个 `env: production` 字段：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "severity": "major",
          "message": "Server unavailable",
          "env": "production",
          "status": 503
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:51:46.795026554Z"
        }
      }
    },
    null,
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "message": "OK",
          "env": "production",
          "status": 200
        },
        "_ingest": {
          "timestamp": "2025-04-23T10:51:46.795048304Z"
        }
      }
    }
  ]
}
```

### 空值安全

使用空安全导航表示法（ `?.` ）来检查字段是否为 `null` 。请注意，这种表示法可能会静默地返回 `null` ；因此，我们建议首先检查返回的值是否为 `null` ，然后使用 `.contains` 或 `==` 等操作。

不安全语法：`"if": "ctx.message?.contains('debug')"`,如果文档中不存在 `message` 字段，此请求将返回一个包含消息 `Cannot invoke "Object.getClass()" because "value" is null` 的 `null_pointer_exception` 。安全语法：`"if": "ctx.message != null && ctx.message.contains('debug')"`

