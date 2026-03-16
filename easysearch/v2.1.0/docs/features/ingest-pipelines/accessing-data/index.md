---
title: "在管道中访问数据"
date: 0001-01-01
summary: "在管道中访问数据 #  在摄取管道中，您可以使用 ctx 对象访问文档数据。此对象表示已处理的文档，并允许您读取、修改或丰富文档字段。管道处理器对文档的 _source 字段及其元数据字段都具有读写访问权限。
访问文档字段 #  ctx 对象公开了所有文档字段。您可以直接使用点符号访问它们。
示例：访问顶级字段 #  给定以下示例文档：
{ &#34;user&#34;: &#34;alice&#34; } 您可以通过以下方式访问 user ：
&#34;field&#34;: &#34;ctx.user&#34; 示例：访问嵌套字段 #  给定以下示例文档：
{ &#34;user&#34;: { &#34;name&#34;: &#34;alice&#34; } } 您可以通过以下方式访问 user.name ：
&#34;field&#34;: &#34;ctx.user.name&#34; 访问源中的字段 #  要访问文档中的字段 _source ，请通过字段名称进行引用：
{ &#34;set&#34;: { &#34;field&#34;: &#34;environment&#34;, &#34;value&#34;: &#34;production&#34; } } 或者，您可以显式使用 _source ：
{ &#34;set&#34;: { &#34;field&#34;: &#34;_source.environment&#34;, &#34;value&#34;: &#34;production&#34; } } 访问元数据字段 #  您可以读取或写入以下元数据字段："
---


# 在管道中访问数据

在摄取管道中，您可以使用 `ctx` 对象访问文档数据。此对象表示已处理的文档，并允许您读取、修改或丰富文档字段。管道处理器对文档的 `_source` 字段及其元数据字段都具有读写访问权限。

## 访问文档字段

`ctx` 对象公开了所有文档字段。您可以直接使用点符号访问它们。

### 示例：访问顶级字段

给定以下示例文档：

```
{
  "user": "alice"
}
```

您可以通过以下方式访问 `user` ：

```
"field": "ctx.user"

```

### 示例：访问嵌套字段

给定以下示例文档：

```
{
  "user": {
    "name": "alice"
  }
}
```

您可以通过以下方式访问 `user.name` ：

```
"field": "ctx.user.name"

```

## 访问源中的字段

要访问文档中的字段 `_source` ，请通过字段名称进行引用：

```
{
  "set": {
    "field": "environment",
    "value": "production"
  }
}
```

或者，您可以显式使用 `_source` ：

```
{
  "set": {
    "field": "_source.environment",
    "value": "production"
  }
}
```

## 访问元数据字段

您可以读取或写入以下元数据字段：

- `_index`
- `_type`
- `_id`
- `_routing`

### 示例：动态设置 `_routing`

```
{
  "set": {
    "field": "_routing",
    "value": "{{region}}"
  }
}
```

## 访问摄取元数据字段

`_ingest.timestamp` 字段表示摄取节点接收文档的时间。要持久化此时间戳，请使用 `set` 处理器：

```
{
  "set": {
    "field": "received_at",
    "value": "{{_ingest.timestamp}}"
  }
}
```

## 使用 `ctx` 在 Mustache 模板中

使用 Mustache 模板将字段值插入处理器设置中。使用三个大括号（ `{{{` 和 `}}}` ）用于未转义的字段值。

### 示例：合并源字段

以下处理器配置将 `app` 和 `env` 字段通过下划线（\_）连接，并将结果存储在 `log_label` 字段中：

```
{
  "set": {
    "field": "log_label",
    "value": "{{{app}}}_{{{env}}}"
  }
}
```

### 示例：使用 `set` 处理器生成动态问候

如果文档的 `user` 字段设置为 `alice` ，请使用以下语法来生成结果 `"greeting": "Hello, alice!"` ：

```
{
  "set": {
    "field": "greeting",
    "value": "Hello, {{{user}}}!"
  }
}
```

## 动态字段名称

您可以使用字段的值作为新字段的名称：

```
{
  "set": {
    "field": "{{service}}",
    "value": "{{code}}"
  }
}
```

### 示例：根据状态路由到动态索引

以下处理器配置通过在 `status` 字段的值后附加 `-events` 来动态设置目标索引：

```
{
  "set": {
    "field": "_index",
    "value": "{{status}}-events"
  }
}
```

## 使用 `ctx` 在 `script` 处理器中

使用 `script` 处理器进行高级转换。

### 示例：如果另一个字段缺失则添加字段

以下处理器仅在文档中缺失字段时，添加具有值“none”的 `error_message` 字段：

```
{
  "script": {
    "lang": "painless",
    "source": "if (ctx.error_message == null) { ctx.error_message = 'none'; }"
  }
}
```

### 示例：将一个字段的值复制到另一个字段

以下处理器将 `timestamp` 字段的值复制到一个名为 `event_time` 的新字段中：

```
{
  "script": {
    "lang": "painless",
    "source": "ctx.event_time = ctx.timestamp;"
  }
}
```

## 示例：完整的管道

以下示例定义了一个完整的摄取管道，该管道使用 `source` 字段设置标语，从 `date` 字段提取 `year` ，并在 `received_at` 字段记录文档的摄取时间戳：

```
PUT _ingest/pipeline/example-pipeline
{
  "description": "Sets tags, log label, and defaults error message",
  "processors": [
    {
      "set": {
        "field": "tagline",
        "value": "{{{user.first}}} from {{{department}}}"
      }
    },
    {
      "script": {
        "lang": "painless",
        "source": "ctx.year = ctx.date.substring(0, 4);"
      }
    },
    {
      "set": {
        "field": "received_at",
        "value": "{{_ingest.timestamp}}"
      }
    }
  ]
}
```

为了测试管道，请使用以下请求：

```
POST _ingest/pipeline/example-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "user": {
          "first": "Liam"
        },
        "department": "Engineering",
        "date": "2024-12-03T14:05:00Z"
      }
    }
  ]
}
```

返回显示了处理后的富文档，包括新添加的 `tagline` ，提取的 `year` ，以及由摄取管道生成的 `received_at` 时间戳：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "user": {
            "first": "Liam"
          },
          "department": "Engineering",
          "date": "2024-12-03T14:05:00Z",
          "tagline": "Liam from Engineering",
          "year": "2024",
          "received_at": "2025-04-14T18:40:00.000Z"
        },
        "_ingest": {
          "timestamp": "2025-04-14T18:40:00.000Z"
        }
      }
    }
  ]
}
```

