---
title: "JSON 处理器"
date: 0001-01-01
summary: "JSON 处理器 #  json 处理器将字符串值字段序列化为一个嵌套的映射，这对于各种数据处理和丰富任务可能很有用。
以下是为 json 处理器提供的语法：
{ &#34;processor&#34;: { &#34;json&#34;: { &#34;field&#34;: &#34;&lt;field_name&gt;&#34;, &#34;target_field&#34;: &#34;&lt;target_field_name&gt;&#34;, &#34;add_to_root&#34;: &lt;boolean&gt; } } } 配置参数 #  下表列出了 json 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要反序列化的 JSON 格式字符串的字段名称。   target_field 可选 字段名称，用于存储反序列化的 JSON 数据。如果未提供，数据将存储在 field 字段中。如果 target_field 存在，其现有值将被新的 JSON 数据覆盖。   add_to_root 可选 一个布尔标志，用于确定是否将反序列化的 JSON 数据添加到文档的根（ true ）或存储在目标字段（ false ）中。如果 add_to_root 是 true ，则 target-field 无效。默认值是 false 。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# JSON 处理器

`json` 处理器将字符串值字段序列化为一个嵌套的映射，这对于各种数据处理和丰富任务可能很有用。


以下是为 `json` 处理器提供的语法：

```
{
  "processor": {
    "json": {
      "field": "<field_name>",
      "target_field": "<target_field_name>",
      "add_to_root": <boolean>
    }
  }
}
```

## 配置参数

下表列出了 `json` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                                                                                  |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 包含要反序列化的 JSON 格式字符串的字段名称。                                                                                                                                          |
| `target_field`   | 可选     | 字段名称，用于存储反序列化的 JSON 数据。如果未提供，数据将存储在 `field` 字段中。如果 `target_field` 存在，其现有值将被新的 JSON 数据覆盖。                                           |
| `add_to_root`    | 可选     | 一个布尔标志，用于确定是否将反序列化的 JSON 数据添加到文档的根（ `true` ）或存储在目标字段（ `false` ）中。如果 `add_to_root` 是 `true` ，则 `target-field` 无效。默认值是 `false` 。 |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                                                                                |
| `if`             | 可选     | 处理器运行的条件。                                                                                                                                                                    |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                                                                                     |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                                                                                      |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                                                                                |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `my-json-pipeline` 的管道，该管道使用 `json` 处理器处理 JSON 数据，并使用附加信息丰富文档：

```
PUT _ingest/pipeline/my-json-pipeline
{
  "description": "Example pipeline using the JsonProcessor",
  "processors": [
    {
      "json": {
        "field": "raw_data",
        "target_field": "parsed_data",
        "on_failure": [
          {
            "set": {
              "field": "error_message",
              "value": "Failed to parse JSON data"
            }
          },
          {
            "fail": {
              "message": "Failed to process JSON data"
            }
          }
        ]
      }
    },
    {
      "set": {
        "field": "processed_timestamp",
        "value": ""
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/my-json-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "raw_data": "{\"name\":\"John\",\"age\":30,\"city\":\"New York\"}"
      }
    },
    {
      "_source": {
        "raw_data": "{\"name\":\"Jane\",\"age\":25,\"city\":\"Los Angeles\"}"
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
          "processed_timestamp": "2024-05-30T15:24:48.064472090Z",
          "raw_data": """{"name":"John","age":30,"city":"New York"}""",
          "parsed_data": {
            "name": "John",
            "city": "New York",
            "age": 30
          }
        },
        "_ingest": {
          "timestamp": "2024-05-30T15:24:48.06447209Z"
        }
      }
    },
    {
      "doc": {
        "_index": "_index",
        "_id": "_id",
        "_source": {
          "processed_timestamp": "2024-05-30T15:24:48.064543006Z",
          "raw_data": """{"name":"Jane","age":25,"city":"Los Angeles"}""",
          "parsed_data": {
            "name": "Jane",
            "city": "Los Angeles",
            "age": 25
          }
        },
        "_ingest": {
          "timestamp": "2024-05-30T15:24:48.064543006Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `my-index` 的索引中：

```
POST my-index/_doc?pipeline=my-json-pipeline
{
  "raw_data": "{\"name\":\"John\",\"age\":30,\"city\":\"New York\"}"
}
```

响应确认，包含来自 `raw_data` 字段的 JSON 数据的文档已成功索引：

```
{
  "_index": "my-index",
  "_id": "mo8yyo8BwFahnwl9WpxG",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 3,
  "_primary_term": 2
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET my-index/_doc/1
```

