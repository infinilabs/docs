---
title: "连接处理器"
date: 0001-01-01
summary: "连接处理器 #  join 处理器将数组中的元素连接成一个单独的字符串值，每个元素之间使用指定的分隔符。如果提供的输入不是数组，则抛出异常。
以下是为 join 处理器提供的语法：
{ &#34;join&#34;: { &#34;field&#34;: &#34;field_name&#34;, &#34;separator&#34;: &#34;separator_string&#34; } } 配置参数 #  下表列出了 join 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 应用连接操作符的字段名称。必须是数组。   separator 必填 字段值连接时使用的字符串分隔符。如果未指定，则值将无分隔符地连接。   target_field 可选 将清理后的值分配给的字段。如果未指定，则字段将就地更新。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 连接处理器

`join` 处理器将数组中的元素连接成一个单独的字符串值，每个元素之间使用指定的分隔符。如果提供的输入不是数组，则抛出异常。


以下是为 `join` 处理器提供的语法：

```
{
  "join": {
    "field": "field_name",
    "separator": "separator_string"
  }
}
```

## 配置参数

下表列出了 `join` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                              |
| ---------------- | -------- | --------------------------------------------------------------------------------- |
| `field`          | 必填     | 应用连接操作符的字段名称。必须是数组。                                            |
| `separator`      | 必填     | 字段值连接时使用的字符串分隔符。如果未指定，则值将无分隔符地连接。                |
| `target_field`   | 可选     | 将清理后的值分配给的字段。如果未指定，则字段将就地更新。                          |
| `description`    | 可选     | 处理器的一个简要描述。                                                            |
| `if`             | 可选     | 处理器运行的条件。                                                                |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。 |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                  |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                            |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `example-join-pipeline` 的管道，该管道使用 `join` 处理器将 `uri` 字段的所有值连接起来，并用指定的分隔符 `/` 分隔：

```
PUT _ingest/pipeline/example-join-pipeline
{
  "description": "Example pipeline using the join processor",
  "processors": [
    {
      "join": {
        "field": "uri",
        "separator": "/"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/example-join-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "uri": [
          "app",
          "home",
          "overview"
        ]
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
          "uri": "app/home/overview"
        },
        "_ingest": {
          "timestamp": "2024-05-24T02:16:01.00659117Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
POST testindex1/_doc/1?pipeline=example-join-pipeline
{
  "uri": [
    "app",
    "home",
    "overview"
  ]
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1

```

