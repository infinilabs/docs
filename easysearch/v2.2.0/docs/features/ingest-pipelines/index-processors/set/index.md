---
title: "设置处理器"
date: 0001-01-01
summary: "设置处理器 #  set 处理器向文档中添加或更新字段。它设置一个字段并将其与指定的值关联。如果该字段已存在，则其值将被提供的值替换，除非设置了 override 参数为 false。当 override 为 false 且指定的字段存在时，该字段的值保持不变。
以下是为 set 处理器提供的语法：
{ &#34;description&#34;: &#34;...&#34;, &#34;processors&#34;: [ { &#34;set&#34;: { &#34;field&#34;: &#34;new_field&#34;, &#34;value&#34;: &#34;some_value&#34; } } ] } 配置参数 #  下表列出了 set 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 要设置或更新的字段的名称。支持模板使用。   value 必填 字段分配的值。支持模板使用。   override 可选 一个布尔标志，用于确定处理器是否应覆盖字段的现有值。   ignore_empty_value 可选 一个布尔标志，用于确定处理器是否应忽略 null 值或空字符串。默认为 false 。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---



# 设置处理器

`set` 处理器向文档中添加或更新字段。它设置一个字段并将其与指定的值关联。如果该字段已存在，则其值将被提供的值替换，除非设置了 `override` 参数为 `false`。当 `override` 为 `false` 且指定的字段存在时，该字段的值保持不变。


以下是为 `set` 处理器提供的语法：

```
{
  "description": "...",
  "processors": [
    {
      "set": {
        "field": "new_field",
        "value": "some_value"
      }
    }
  ]
}
```

## 配置参数

下表列出了 `set` 处理器所需的和可选参数。

| 参数                 | 是否必填 | 描述                                                                              |
| -------------------- | -------- | --------------------------------------------------------------------------------- |
| `field`              | 必填     | 要设置或更新的字段的名称。支持模板使用。                                          |
| `value`              | 必填     | 字段分配的值。支持模板使用。                                                      |
| `override`           | 可选     | 一个布尔标志，用于确定处理器是否应覆盖字段的现有值。                              |
| `ignore_empty_value` | 可选     | 一个布尔标志，用于确定处理器是否应忽略 null 值或空字符串。默认为 false 。         |
| `description`        | 可选     | 处理器的一个简要描述。                                                            |
| `if`                 | 可选     | 处理器运行的条件。                                                                |
| `ignore_failure`     | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。 |
| `on_failure`         | 可选     | 在处理器失败时运行的处理器列表。                                                  |
| `tag`                | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                            |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `set-pipeline` 的管道，该管道使用 `set` 处理器向文档添加一个新字段 `new_field` ，其值为 `some_value` ：

```
PUT _ingest/pipeline/set-pipeline
{
  "description": "Adds a new field 'new_field' with the value 'some_value'",
  "processors": [
    {
      "set": {
        "field": "new_field",
        "value": "some_value"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/set-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "existing_field": "value"
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
          "existing_field": "value",
          "new_field": "some_value"
        },
        "_ingest": {
          "timestamp": "2024-05-30T21:56:15.066180712Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
POST testindex1/_doc/1?pipeline=set-pipeline
{
  "existing_field": "value"
}
```

请求将文档索引到索引 `testindex1` 中，然后索引所有将 `new_field` 设置为 `some_value` 的文档，如下面的响应所示：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

