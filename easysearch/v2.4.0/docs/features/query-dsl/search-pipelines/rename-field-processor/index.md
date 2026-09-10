---
title: "重命名字段处理器"
date: 0001-01-01
summary: "重命名字段处理器（Rename Field Processor） #  版本引入：1.14.0
rename_field 结果增强处理器用于拦截搜索响应，并对指定字段进行重命名。当你索引中的字段名称与应用程序使用的名称不一致时，该功能非常有用。例如，当索引中使用了新的字段名称，但应用程序仍期望接收旧字段名称时，可通过 rename_field 处理器在返回响应前完成字段名的映射，实现平滑过渡和向后兼容。
请求体字段 #  下表列出了该处理器支持的所有配置字段。
   字段 数据类型 说明     field 字符串 要重命名的原始字段名。必填。   target_field 字符串 新的字段名称。必填。   tag 字符串 处理器的唯一标识符。可选。   description 字符串 对该处理器的描述信息。可选。   ignore_failure 布尔值 若为 true，当此处理器执行失败时，Easysearch 将忽略错误并继续执行搜索管道中的其余处理器。可选，默认值为 false。    示例 #  以下示例演示如何在搜索管道中使用 rename_field 处理器。
准备工作 #  创建一个名为 my_index 的索引，并索引一个包含 message 字段的文档：
POST /my_index/_doc/1 { &#34;message&#34;: &#34;This is a public message&#34;, &#34;visibility&#34;: &#34;public&#34; } 创建搜索管道 #  以下请求创建一个名为 my_pipeline 的搜索管道，其中包含一个 rename_field 结果增强处理器，用于将字段 message 重命名为 notification："
---


# 重命名字段处理器（Rename Field Processor）
版本引入：1.14.0

`rename_field` 结果增强处理器用于拦截搜索响应，并对指定字段进行重命名。当你索引中的字段名称与应用程序使用的名称不一致时，该功能非常有用。例如，当索引中使用了新的字段名称，但应用程序仍期望接收旧字段名称时，可通过 `rename_field` 处理器在返回响应前完成字段名的映射，实现平滑过渡和向后兼容。

## 请求体字段

下表列出了该处理器支持的所有配置字段。

| 字段 | 数据类型 | 说明                                                                     |
| :--- | :--- |:-----------------------------------------------------------------------|
| `field` | 字符串 | 要重命名的原始字段名。**必填**。                                                     |
| `target_field` | 字符串 | 新的字段名称。**必填**。                                                         |
| `tag` | 字符串 | 处理器的唯一标识符。可选。                                                          |
| `description` | 字符串 | 对该处理器的描述信息。可选。                                                         |
| `ignore_failure` | 布尔值 | 若为 `true`，当此处理器执行失败时，Easysearch 将忽略错误并继续执行搜索管道中的其余处理器。可选，默认值为 `false`。 |

## 示例

以下示例演示如何在搜索管道中使用 `rename_field` 处理器。

### 准备工作

创建一个名为 `my_index` 的索引，并索引一个包含 `message` 字段的文档：

```json
POST /my_index/_doc/1
{
  "message": "This is a public message",
  "visibility": "public"
}
```

### 创建搜索管道

以下请求创建一个名为 `my_pipeline` 的搜索管道，其中包含一个 `rename_field` 结果增强处理器，用于将字段 `message` 重命名为 `notification`：

```auto
PUT /_search/pipeline/my_pipeline
{
  "enrich_processors": [
    {
      "rename_field": {
        "field": "message",
        "target_field": "notification"
      }
    }
  ]
}
```

### 使用搜索管道

在不使用搜索管道的情况下，对 `my_index` 索引执行搜索：

```auto
GET /my_index/_search
```

响应中包含原始字段 `message`：

<details open markdown="block">
  <summary>
    响应
  </summary>

```auto
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "my_index",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "message": "This is a public message",
          "visibility": "public"
        }
      }
    ]
  }
}
```
</details>

要使用搜索管道，请在请求中通过 `search_pipeline` 查询参数指定管道名称：

```auto
GET /my_index/_search?search_pipeline=my_pipeline
```

此时，字段 `message` 已被重命名为 `notification`：

<details open markdown="block">
  <summary>
    响应
  </summary>

```auto
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.0,
    "hits": [
      {
        "_index": "my_index",
        "_id": "1",
        "_score": 0.0,
        "_source": {
          "visibility": "public",
          "notification": "This is a public message"
        }
      }
    ]
  }
}
```
</details>

你也可以使用 `fields` 参数来搜索文档中的特定字段：

```auto
POST /my_index/_search?pretty&search_pipeline=my_pipeline
{
    "fields": ["visibility", "message"]
}
```

在响应中，`message` 字段已被重命名为 `notification`，包括在 `fields` 返回结果中：

<details open markdown="block">
  <summary>
    响应
  </summary>

```auto
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.0,
    "hits": [
      {
        "_index": "my_index",
        "_id": "1",
        "_score": 0.0,
        "_source": {
          "visibility": "public",
          "notification": "This is a public message"
        },
        "fields": {
          "visibility": [
            "public"
          ],
          "notification": [
            "This is a public message"
          ]
        }
      }
    ]
  }
}
```
</details>

