---
title: "大写处理器"
date: 0001-01-01
summary: "大写处理器 #  uppercase 处理器将特定字段中的所有文本转换为大写字母。
语法 #  以下是为 uppercase 处理器提供的语法：
{ &#34;uppercase&#34;: { &#34;field&#34;: &#34;field_name&#34; } } 配置参数 #  下表列出了 uppercase 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要附加数据的字段名称。支持模板使用。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。   target_field 可选 要存储解析数据的字段名称。默认为 field 。默认情况下， field 将就地更新。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 大写处理器

`uppercase` 处理器将特定字段中的所有文本转换为大写字母。


## 语法

以下是为 `uppercase` 处理器提供的语法：

```
{
  "uppercase": {
    "field": "field_name"
  }
}
```

## 配置参数

下表列出了 `uppercase` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                      |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 包含要附加数据的字段名称。支持模板使用。                                                                                  |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                    |
| `if`             | 可选     | 处理器运行的条件。                                                                                                        |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                         |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。 |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                          |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                    |
| `target_field`   | 可选     | 要存储解析数据的字段名称。默认为 field 。默认情况下， field 将就地更新。                                                  |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `uppercase` 的管道，该管道将 `field` 字段中的文本转换为大写：

```
PUT _ingest/pipeline/uppercase
{
  "processors": [
    {
      "uppercase": {
        "field": "name"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/uppercase/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "name": "John"
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
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "name": "JOHN"
        },
        "_ingest": {
          "timestamp": "2023-08-28T19:54:42.289624792Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=uppercase
{
  "name": "John"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

