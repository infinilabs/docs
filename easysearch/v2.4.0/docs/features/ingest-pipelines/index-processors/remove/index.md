---
title: "移除处理器"
date: 0001-01-01
summary: "移除处理器 #  remove 处理器用于从文档中移除一个字段。
语法 #  以下是为 remove 处理器提供的语法：
{ &#34;remove&#34;: { &#34;field&#34;: &#34;field_name&#34; } } 配置参数 #  下表列出了 remove 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要移除的数据的字段名称。支持模板片段。 _index 、 _version 、 _version_type 和 _id 元数据字段不能被移除。如果指定了 version ，则不能从摄取文档中移除 _id 。   exclude_field 必填 要保留的字段名称。除元数据字段外，所有其他字段都将被删除。 exclude_field 和 field 选项互斥。支持模板片段。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 移除处理器

`remove` 处理器用于从文档中移除一个字段。


## 语法

以下是为 `remove` 处理器提供的语法：

```
{
    "remove": {
        "field": "field_name"
    }
}
```

## 配置参数

下表列出了 `remove` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                                                                       |
| ---------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 包含要移除的数据的字段名称。支持模板片段。 `_index` 、 `_version` 、 `_version_type` 和 `_id` 元数据字段不能被移除。如果指定了 `version` ，则不能从摄取文档中移除 `_id` 。 |
| `exclude_field`  | 必填     | 要保留的字段名称。除元数据字段外，所有其他字段都将被删除。 `exclude_field` 和 `field` 选项互斥。支持模板片段。                                                             |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                                                                     |
| `if`             | 可选     | 处理器运行的条件。                                                                                                                                                         |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                                                          |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                                                                           |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                                                                     |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `remove_ip` 的管道，用于从文档中删除 `ip_address` 字段：

```
PUT /_ingest/pipeline/remove_ip
{
  "description": "Pipeline that excludes the ip_address field.",
  "processors": [
    {
      "remove": {
        "field": "ip_address"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/remove_ip/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source":{
         "ip_address": "203.0.113.1",
         "name": "John Doe"
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
          "name": "John Doe"
        },
        "_ingest": {
          "timestamp": "2023-08-24T18:02:13.218986756Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=remove_ip
{
  "ip_address": "203.0.113.1",
  "name": "John Doe"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

