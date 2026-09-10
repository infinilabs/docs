---
title: "字节处理器"
date: 0001-01-01
summary: "字节处理器 #  bytes 处理器将可读的字节值转换为等效的字节数值。字段可以是标量或数组。如果字段是标量，则值将被转换并存储在该字段中。如果字段是数组，则转换数组中的所有值。
语法 #  以下是为 bytes 处理器提供的语法：
{ &#34;bytes&#34;: { &#34;field&#34;: &#34;your_field_name&#34; } } 配置参数 #  下表列出了 bytes 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要转换的数据的字段名称。支持模板使用。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。   target_field 可选 指定存储解析数据的字段名称。如果没有指定，则值将存储在 field 字段中。默认为 field 。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 字节处理器

`bytes` 处理器将可读的字节值转换为等效的字节数值。字段可以是标量或数组。如果字段是标量，则值将被转换并存储在该字段中。如果字段是数组，则转换数组中的所有值。


## 语法

以下是为 `bytes` 处理器提供的语法：

```
{
    "bytes": {
        "field": "your_field_name"
    }
}
```

## 配置参数

下表列出了 `bytes` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                      |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 包含要转换的数据的字段名称。支持模板使用。                                                                                |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                    |
| `if`             | 可选     | 处理器运行的条件。                                                                                                        |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                         |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。 |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                          |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                    |
| `target_field`   | 可选     | 指定存储解析数据的字段名称。如果没有指定，则值将存储在 `field` 字段中。默认为 `field` 。                                  |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `file_upload` 的管道，该管道有一个 `bytes` 处理器。它将 `file_size` 转换为字节的等效值，并将其存储在名为 `file_size_bytes` 的新字段中：

```
PUT _ingest/pipeline/file_upload
{
  "description": "Pipeline that converts file size to bytes",
  "processors": [
    {
      "bytes": {
        "field": "file_size",
        "target_field": "file_size_bytes"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/file_upload/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "file_size_bytes": "10485760",
        "file_size":
          "10MB"
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
          "file_size_bytes": "10485760",
          "file_size": "10MB"
        },
        "_ingest": {
          "timestamp": "2023-08-22T16:09:42.771569211Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=file_upload
{
  "file_size": "10MB"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

