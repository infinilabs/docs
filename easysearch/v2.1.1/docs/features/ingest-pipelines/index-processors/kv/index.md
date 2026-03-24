---
title: "KV 处理器"
date: 0001-01-01
summary: "KV 处理器 #  kv 处理器会自动提取特定事件字段或消息，这些字段或消息以 key=value 格式存在。这种结构化格式通过将数据根据键和值分组来组织您的数据。这对于分析、可视化和使用数据非常有帮助，例如用户行为分析、性能优化或安全调查。
以下是为 kv 处理器提供的语法：
{ &#34;kv&#34;: { &#34;field&#34;: &#34;message&#34;, &#34;field_split&#34;: &#34; &#34;, &#34;value_split&#34;: &#34; &#34; } } 配置参数 #  下表列出了 kv 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要解析的数据的字段名称。   field_split 必填 键值对分割的正则表达式模式。   value_split 必填 从键值对中分割键和值的正则表达式模式，例如，等号 = 或冒号 : 。   exclude_keys 可选 要排除的文档中的键。默认为 null 。   include_keys 可选 用于过滤和插入的键。默认为包含所有键。   prefix 可选 添加到提取键的前缀。默认为 null 。   strip_brackets 可选 如果设置为 true ，则从提取的值中去除括号（ () 、 &lt;&gt;, 或 [] ）和引号（ ' 或 &quot; ）。默认为 false 。   trim_key 可选 从提取键中修剪的字符字符串。   trim_value 可选 从提取值中修剪的字符字符串。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。默认为 false 。   target_field 可选 要插入提取的键的字段名称。默认为 null 。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# KV 处理器

`kv` 处理器会自动提取特定事件字段或消息，这些字段或消息以 `key=value` 格式存在。这种结构化格式通过将数据根据键和值分组来组织您的数据。这对于分析、可视化和使用数据非常有帮助，例如用户行为分析、性能优化或安全调查。


以下是为 `kv` 处理器提供的语法：

```
{
  "kv": {
    "field": "message",
    "field_split": " ",
    "value_split": " "
  }
}
```

## 配置参数

下表列出了 `kv` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                       |
| ---------------- | -------- | ---------------------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 包含要解析的数据的字段名称。                                                                               |
| `field_split`    | 必填     | 键值对分割的正则表达式模式。                                                                               |
| `value_split`    | 必填     | 从键值对中分割键和值的正则表达式模式，例如，等号 `=` 或冒号 `:` 。                                         |
| `exclude_keys`   | 可选     | 要排除的文档中的键。默认为 null 。                                                                         |
| `include_keys`   | 可选     | 用于过滤和插入的键。默认为包含所有键。                                                                     |
| `prefix`         | 可选     | 添加到提取键的前缀。默认为 null 。                                                                         |
| `strip_brackets` | 可选     | 如果设置为 true ，则从提取的值中去除括号（ `()` 、 `<>`, 或 `[]` ）和引号（ `'` 或 `"` ）。默认为 false 。 |
| `trim_key`       | 可选     | 从提取键中修剪的字符字符串。                                                                               |
| `trim_value`     | 可选     | 从提取值中修剪的字符字符串。                                                                               |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                     |
| `if`             | 可选     | 处理器运行的条件。                                                                                         |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                          |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                           |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定字段的文档。默认为 false 。                                                  |
| `target_field`   | 可选     | 要插入提取的键的字段名称。默认为 null 。                                                                   |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                     |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `kv-pipeline` 的管道，该管道使用 `kv` 处理器提取文档的 `message` 字段：

```
PUT _ingest/pipeline/kv-pipeline
{
  "description" : "Pipeline that extracts user profile data",
  "processors" : [
    {
      "kv" : {
        "field" : "message",
        "field_split": " ",
        "value_split": "="
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/kv-pipeline/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source":{
         "message": "goodbye=everybody hello=world"
      }
    }
  ]
}
```

以下示例响应确认，除了原始的 `message` 字段外，该文档还包含由键值对生成的字段：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "hello": "world",
          "message": "goodbye=everybody hello=world",
          "goodbye": "everybody"
        },
        "_ingest": {
          "timestamp": "2023-12-06T09:59:21.823292Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=kv-pipeline
{
  "message": "goodbye=everybody hello=world"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

