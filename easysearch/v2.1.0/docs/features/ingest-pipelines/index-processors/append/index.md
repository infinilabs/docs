---
title: "添加处理器"
date: 0001-01-01
summary: "添加处理器 #  append 处理器用于向字段添加值：
 如果该字段是数组， append 处理器会将指定的值追加到该数组中。 如果该字段是标量字段， append 处理器将其转换为数组，并将指定的值追加到该数组中。 如果该字段不存在， append 处理器会创建一个包含指定值的数组。  语法 #  以下是为 append 处理器提供的语法：
{ &#34;append&#34;: { &#34;field&#34;: &#34;your_target_field&#34;, &#34;value&#34;: [&#34;your_appended_value&#34;] } } 配置参数 #  下表列出了 append 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要附加数据的字段名称。支持模板使用。   value 必填 要附加的值。这可以是一个静态值或从现有字段派生的动态值。支持模板使用。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   allow_duplicates 可选 指定是否将字段中已存在的值附加。如果为 true ，则附加重复值。否则，将跳过。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 添加处理器

`append` 处理器用于向字段添加值：


- 如果该字段是数组， `append` 处理器会将指定的值追加到该数组中。
- 如果该字段是标量字段， `append` 处理器将其转换为数组，并将指定的值追加到该数组中。
- 如果该字段不存在， `append` 处理器会创建一个包含指定值的数组。

## 语法

以下是为 `append` 处理器提供的语法：

```
{
  "append": {
    "field": "your_target_field",
    "value": ["your_appended_value"]
  }
}
```

## 配置参数

下表列出了 `append` 处理器所需的和可选参数。

| 参数               | 是否必填 | 描述                                                                              |
| ------------------ | -------- | --------------------------------------------------------------------------------- |
| `field`            | 必填     | 包含要附加数据的字段名称。支持模板使用。                                          |
| `value`            | 必填     | 要附加的值。这可以是一个静态值或从现有字段派生的动态值。支持模板使用。            |
| `description`      | 可选     | 处理器的一个简要描述。                                                            |
| `if`               | 可选     | 处理器运行的条件。                                                                |
| `ignore_failure`   | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。 |
| `allow_duplicates` | 可选     | 指定是否将字段中已存在的值附加。如果为 true ，则附加重复值。否则，将跳过。        |
| `on_failure`       | 可选     | 在处理器失败时运行的处理器列表。                                                  |
| `tag`              | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                            |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `user-behavior` 的管道，该管道包含一个追加处理器。它将每个新文档的 `page_view` 追加到名为 `event_types` 的数组字段中：

```
PUT _ingest/pipeline/user-behavior
{
  "description": "Pipeline that appends event type",
  "processors": [
    {
      "append": {
        "field": "event_types",
        "value": ["page_view"]
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/user-behavior/_simulate
{
  "docs":[
    {
      "_source":{
      }
    }
  ]
}
```

返回的内容确认管道按预期工作：

```
{
  "docs": [
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "event_types": [
            "page_view"
          ]
        },
        "_ingest": {
          "timestamp": "2023-08-28T16:55:10.621805166Z"
        }
      }
    }
  ]
}

```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=user-behavior
{
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

由于文档不包含 `event_types` 字段，将创建一个数组字段并将事件追加到该数组中：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 2,
  "_seq_no": 1,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "event_types": [
      "page_view"
    ]
  }
}
```

