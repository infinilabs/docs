---
title: "Sort 处理器"
date: 0001-01-01
summary: "Sort 处理器 #  sort 处理器可以按升序或降序对项目数组进行排序。数值数组按数值排序，而字符串或混合数组（字符串和数字）按字典顺序排序。如果输入不是数组，处理器将抛出错误。
以下是为 sort 处理器提供的语法：
{ &#34;description&#34;: &#34;Sort an array of items&#34;, &#34;processors&#34;: [ { &#34;sort&#34;: { &#34;field&#34;: &#34;my_array_field&#34;, &#34;order&#34;: &#34;desc&#34; } } ] } 配置参数 #  下表列出了 sort 处理器所需的和可选参数。
   参数 是否必填 描述     field 必填 要排序的字段。必须是数组。   order 可选 应用排序顺序。接受 asc 用于升序或 desc 用于降序。默认是 asc 。   target_field 可选 存储排序数组字段的名称。如果未指定，则排序数组将存储在原始数组（ field 变量）相同的字段中。   description 可选 处理器的一个简要描述。   if 可选 处理器运行的条件。   ignore_failure 可选 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。   on_failure 可选 在处理器失败时运行的处理器列表。   tag 可选 处理器的标识标签。在调试中区分同一类型的处理器很有用。    如何使用 #  按照以下步骤在管道中使用处理器。"
---


# Sort 处理器

`sort` 处理器可以按升序或降序对项目数组进行排序。数值数组按数值排序，而字符串或混合数组（字符串和数字）按字典顺序排序。如果输入不是数组，处理器将抛出错误。


以下是为 `sort` 处理器提供的语法：

```
{
  "description": "Sort an array of items",
  "processors": [
    {
      "sort": {
        "field": "my_array_field",
        "order": "desc"
      }
    }
  ]
}
```

## 配置参数

下表列出了 `sort` 处理器所需的和可选参数。

| 参数             | 是否必填 | 描述                                                                                          |
| ---------------- | -------- | --------------------------------------------------------------------------------------------- |
| `field`          | 必填     | 要排序的字段。必须是数组。                                                                    |
| `order`          | 可选     | 应用排序顺序。接受 `asc` 用于升序或 `desc` 用于降序。默认是 `asc` 。                          |
| `target_field`   | 可选     | 存储排序数组字段的名称。如果未指定，则排序数组将存储在原始数组（ `field` 变量）相同的字段中。 |
| `description`    | 可选     | 处理器的一个简要描述。                                                                        |
| `if`             | 可选     | 处理器运行的条件。                                                                            |
| `ignore_failure` | 可选     | 指定处理器是否在遇到错误时继续执行。如果设置为 true ，则忽略失败。默认为 false 。             |
| `on_failure`     | 可选     | 在处理器失败时运行的处理器列表。                                                              |
| `tag`            | 可选     | 处理器的标识标签。在调试中区分同一类型的处理器很有用。                                        |

## 如何使用

按照以下步骤在管道中使用处理器。

### 步骤 1：创建一个管道

以下查询创建了一个名为 `sort-pipeline` 的管道，使用 `sort` 处理器按降序对 `my_field` 进行排序，并将排序后的值存储在 `sorted_field` 中：

```
PUT _ingest/pipeline/sort-pipeline
{
  "description": "Sort an array of items in descending order",
  "processors": [
    {
      "sort": {
        "field": "my_array_field",
        "order": "desc",
        "target_field": "sorted_array"
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

> 建议您在摄取文档之前测试您的管道。

要测试管道，请运行以下查询：

```
POST _ingest/pipeline/sort-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "my_array_field": [3, 1, 4, 1, 5, 9, 2, 6, 5]
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
          "sorted_array": [
            9,
            6,
            5,
            5,
            4,
            3,
            2,
            1,
            1
          ],
          "my_array_field": [
            3,
            1,
            4,
            1,
            5,
            9,
            2,
            6,
            5
          ]
        },
        "_ingest": {
          "timestamp": "2024-05-30T22:10:13.405692128Z"
        }
      }
    }
  ]
}
```

### 步骤 3：摄取文档

以下查询将文档索引到名为 `testindex1` 的索引中：

```
POST testindex1/_doc/1?pipeline=sort-pipeline
{
  "my_array_field": [3, 1, 4, 1, 5, 9, 2, 6, 5]
}
```

请求将文档索引到索引 `testindex1` 中，然后按降序索引所有 `my_array_field` 文档，如下面的响应所示：

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
  "_seq_no": 9,
  "_primary_term": 2
}
```

### 步骤 4（可选）：检索文档

要检索文档，请运行以下查询：

```
GET testindex1/_doc/1
```

