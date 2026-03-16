---
title: "重命名处理器"
date: 0001-01-01
summary: "重命名处理器 #  rename 处理器用于重命名现有字段，也可以用来将字段从一个对象移动到另一个对象或根级别。
语法 #  以下是为 rename 处理器提供的语法：
{ &#34;rename&#34;: { &#34;field&#34;: &#34;field_name&#34;, &#34;target_field&#34; : &#34;target_field_name&#34; } } 配置参数 #  下表列出了 rename 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要重命名的数据的字段名称。支持模板使用。   target_field 必填 字段的新名称。支持模板使用。   ignore_missing 可选 指定处理器是否应忽略不包含指定 field 的文档。如果设置为 true ，则处理器在 field 不存在时不会修改文档。默认为 false 。   override_target 可选 确定当 target_field 存在于文档中时会发生什么。如果设置为 true ，则处理器将现有的 target_field 值覆盖为新值。如果设置为 false ，则现有值保持不变，处理器不会覆盖它。默认为 false 。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 重命名处理器

`rename` 处理器用于重命名现有字段，也可以用来将字段从一个对象移动到另一个对象或根级别。


## 语法

以下是为 `rename` 处理器提供的语法：

```
{
    "rename": {
        "field": "field_name",
        "target_field" : "target_field_name"
    }
}
```

## 配置参数

下表列出了 `rename` 处理器所需的和可选参数。

| 参数              | 是否必填 | 描述                                                                                                                                                                                         |
| ----------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`           | 必填     | 包含要重命名的数据的字段名称。支持模板使用。                                                                                                                                                 |
| `target_field`    | 必填     | 字段的新名称。支持模板使用。                                                                                                                                                                 |
| `ignore_missing`  | 可选     | 指定处理器是否应忽略不包含指定 `field` 的文档。如果设置为 `true` ，则处理器在 `field` 不存在时不会修改文档。默认为 `false` 。                                                                |
| `override_target` | 可选     | 确定当 `target_field` 存在于文档中时会发生什么。如果设置为 `true` ，则处理器将现有的 `target_field` 值覆盖为新值。如果设置为 `false` ，则现有值保持不变，处理器不会覆盖它。默认为 `false` 。 |
| `description`     | 可选     | 处理器的一个简要描述。                                                                                                                                                                       |
| `if`              | 可选     | 处理器运行的条件。                                                                                                                                                                           |
| `ignore_failure`  | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                                                                            |
| `on_failure`      | 可选     | 在处理器失败时运行的处理器列表。                                                                                                                                                             |
| `tag`             | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                                                                                       |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `rename_field` 的管道，该管道将对象中的一个字段移动到根级别：

```
PUT /_ingest/pipeline/rename_field
{
  "description": "Pipeline that moves a field to the root level.",
  "processors": [
    {
      "rename": {
        "field": "message.content",
        "target_field": "content"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/rename_field/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source":{
         "message": {
           "type": "nginx",
           "content": "192.168.1.10 - - [03/Nov/2023:15:20:45 +0000] \"POST /login HTTP/1.1\" 200 3456"
         }
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
          "message": {
            "type": "nginx",
          },
          "content": """192.168.1.10 - - [03/Nov/2023:15:20:45 +0000] "POST /login HTTP/1.1" 200 3456"""
        },
        "_ingest": {
          "timestamp": "2024-04-15T07:54:16.010447Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=rename_field
{
  "message": {
    "type": "nginx",
    "content": "192.168.1.10 - - [03/Nov/2023:15:20:45 +0000] \"POST /login HTTP/1.1\" 200 3456"
  }
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

