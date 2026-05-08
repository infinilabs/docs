---
title: "CSV 处理器"
date: 0001-01-01
summary: "CSV 处理器 #  csv 处理器用于解析 CSV 文件并将它们作为单独的字段存储在文档中。该处理器会忽略空字段。
语法 #  以下是为 csv 处理器提供的语法：
{ &#34;csv&#34;: { &#34;field&#34;: &#34;field_name&#34;, &#34;target_fields&#34;: [&#34;field1, field2, ...&#34;] } } 配置参数 #  下表列出了 csv 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 包含要转换的数据的字段名称。支持模板使用。   target_fields 必填 字段名称，用于存储解析后的数据。   description 可选 处理器的一个简要描述。   empty_value 可选 表示可选参数，这些参数不是必需的或不适用的。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   ignore_missing 可选 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   quote 可选 用于在 CSV 数据中引用字段的字符。默认为 &quot; 。   separator 可选 用于在 CSV 数据中分隔字段的分隔符。默认为 , 。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。   trim 可选 如果设置为 true ，处理器将删除文本开头和结尾的空白字符。默认是 false 。    如何使用 #  按照以下步骤在管道中使用处理器。"
---



# CSV 处理器

`csv` 处理器用于解析 CSV 文件并将它们作为单独的字段存储在文档中。该处理器会忽略空字段。


## 语法

以下是为 `csv` 处理器提供的语法：

```
{
  "csv": {
    "field": "field_name",
    "target_fields": ["field1, field2, ..."]
  }
}
```

## 配置参数

下表列出了 `csv` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                                                      |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 包含要转换的数据的字段名称。支持模板使用。                                                                                |
| `target_fields`  | 必填     | 字段名称，用于存储解析后的数据。                                                                                          |
| `description`    | 可选     | 处理器的一个简要描述。                                                                                                    |
| `empty_value`    | 可选     | 表示可选参数，这些参数不是必需的或不适用的。                                                                              |
| `if`             | 可选     | 处理器运行的条件。                                                                                                        |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。                                         |
| `ignore_missing` | 可选     | 指定处理器是否应忽略不包含指定字段的文档。如果设置为 true ，则处理器在字段不存在或为 null 时不会修改文档。默认为 false 。 |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                                                          |
| `quote`          | 可选     | 用于在 CSV 数据中引用字段的字符。默认为 `"` 。                                                                            |
| `separator`      | 可选     | 用于在 CSV 数据中分隔字段的分隔符。默认为 `,` 。                                                                          |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                                                    |
| `trim`           | 可选     | 如果设置为 true ，处理器将删除文本开头和结尾的空白字符。默认是 false 。                                                   |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `csv-processor` 的管道，将 `resource_usage` 拆分为三个新字段，分别命名为 `cpu_usage` 、 `memory_usage` 和 `disk_usage` ：

```
PUT _ingest/pipeline/csv-processor
{
  "description": "Split resource usage into individual fields",
  "processors": [
    {
      "csv": {
        "field": "resource_usage",
        "target_fields": ["cpu_usage", "memory_usage", "disk_usage"],
        "separator": ","
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/csv-processor/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "resource_usage": "25,4096,10",
        "memory_usage": "4096",
        "disk_usage": "10",
        "cpu_usage": "25"
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
          "memory_usage": "4096",
          "disk_usage": "10",
          "resource_usage": "25,4096,10",
          "cpu_usage": "25"
        },
        "_ingest": {
          "timestamp": "2023-08-22T16:40:45.024796379Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
PUT testindex1/_doc/1?pipeline=csv-processor
{
  "resource_usage": "25,4096,10"
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

