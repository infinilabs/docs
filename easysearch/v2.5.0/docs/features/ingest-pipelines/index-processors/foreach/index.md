---
title: "遍历处理器"
date: 0001-01-01
summary: "遍历处理器 #  foreach 处理器用于遍历输入文档中的值列表并对每个值应用转换。这可以用于处理数组中的所有元素，例如将字符串中的所有元素转换为小写或大写等任务。
以下是为 foreach 处理器提供的语法：
{ &#34;foreach&#34;: { &#34;field&#34;: &#34;&lt;field_name&gt;&#34;, &#34;processor&#34;: { &#34;&lt;processor_type&gt;&#34;: { &#34;&lt;processor_config&gt;&#34;: &#34;&lt;processor_value&gt;&#34; } } } } 配置参数 #  下表列出了 foreach 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 要遍历的数组字段。   processor 必填 处理器用于对每个字段执行操作。   ignore_missing 可选 如果 true 指定的字段不存在或为 null，则处理器将静默退出，不会修改文档。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# 遍历处理器

`foreach` 处理器用于遍历输入文档中的值列表并对每个值应用转换。这可以用于处理数组中的所有元素，例如将字符串中的所有元素转换为小写或大写等任务。


以下是为 `foreach` 处理器提供的语法：

```
{
  "foreach": {
    "field": "<field_name>",
    "processor": {
      "<processor_type>": {
        "<processor_config>": "<processor_value>"
      }
    }
  }
}
```

## 配置参数

下表列出了 `foreach` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                              |
| ---------------- | -------- | --------------------------------------------------------------------------------- |
| `field`          | 必填     | 要遍历的数组字段。                                                                |
| `processor`      | 必填     | 处理器用于对每个字段执行操作。                                                    |
| `ignore_missing` | 可选     | 如果 `true` 指定的字段不存在或为 null，则处理器将静默退出，不会修改文档。         |
| `description`    | 可选     | 处理器的一个简要描述。                                                            |
| `if`             | 可选     | 处理器运行的条件。                                                                |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。 |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                  |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                            |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建管道

以下查询创建了一个名为 `test-foreach` 的管道，该管道使用 `foreach` 处理器遍历 `protocols` 字段中的每个元素：

```
PUT _ingest/pipeline/test-foreach
{
  "description": "Lowercase all the elements in an array",
  "processors": [
    {
      "foreach": {
        "field": "protocols",
        "processor": {
          "lowercase": {
            "field": "_ingest._value"
          }
        }
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/test-foreach/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "protocols": ["HTTP","HTTPS","TCP","UDP"]
      }
    }
  ]
}
```

以下示例响应确认管道按预期工作，显示四个元素已被转换为小写：

```
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "protocols": [
            "http",
            "https",
            "tcp",
            "udp"
          ]
        },
        "_ingest": {
          "_value": null,
          "timestamp": "2024-05-23T02:44:10.8201Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
POST testindex1/_doc/1?pipeline=test-foreach
{
  "protocols": ["HTTP","HTTPS","TCP","UDP"]
}
```

请求将文档索引到索引 `testindex1` ，并在索引前应用管道：

```
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 6,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 5,
  "_primary_term": 67
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

显示从 users 字段提取的 JSON 数据文档：

```
{
  "_index": "testindex1",
  "_type": "_doc",
  "_id": "1",
  "_version": 8,
  "_seq_no": 7,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "protocols": [
      "http",
      "https",
      "tcp",
      "udp"
    ]
  }
}
```

