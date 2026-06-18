---
title: "过滤查询处理器"
date: 0001-01-01
summary: "过滤查询处理器（filter query processor） #  版本引入：1.14.0
filter_query 查询重写处理器用于拦截搜索请求，并向该请求中添加一个额外的查询条件，从而对搜索结果进行过滤。当你不希望重写应用程序中已有的查询语句，但又需要对结果进行额外过滤时，此功能非常有用。
请求体字段 #  下表列出了所有可用的请求字段。
   字段 数据类型 说明     query 对象 使用 Easysearch 查询领域特定语言（DSL）编写的查询语句。必填。   tag 字符串 处理器的唯一标识符。可选。   description 字符串 对该处理器的描述信息。可选。   ignore_failure 布尔值 若为 true，则当此处理器执行失败时，Easysearch 将忽略该错误并继续执行搜索管道中的其余处理器。可选，默认值为 false。    示例 #  以下示例演示如何在搜索管道中使用 filter_query 处理器。
准备工作 #  创建一个名为 my_index 的索引，并索引两个文档：一个公开，一个私有：
POST /my_index/_doc/1 { &#34;message&#34;: &#34;This is a public message&#34;, &#34;visibility&#34;: &#34;public&#34; } POST /my_index/_doc/2 { &#34;message&#34;: &#34;This is a private message&#34;, &#34;visibility&#34;: &#34;private&#34; } 创建搜索管道 #  以下请求创建一个名为 my_pipeline 的搜索管道，其中包含一个 filter_query 查询重写处理器。该处理器使用 term 查询，仅返回可见性为“public”的文档："
---


# 过滤查询处理器（filter query processor）
版本引入：1.14.0

`filter_query` 查询重写处理器用于拦截搜索请求，并向该请求中添加一个额外的查询条件，从而对搜索结果进行过滤。当你不希望重写应用程序中已有的查询语句，但又需要对结果进行额外过滤时，此功能非常有用。

## 请求体字段

下表列出了所有可用的请求字段。

| 字段 | 数据类型 | 说明                                                                       |
| :--- | :--- |:-------------------------------------------------------------------------|
| `query` | 对象 | 使用 Easysearch 查询领域特定语言（DSL）编写的查询语句。**必填**。                               |
| `tag` | 字符串 | 处理器的唯一标识符。可选。                                                            |
| `description` | 字符串 | 对该处理器的描述信息。可选。                                                           |
| `ignore_failure` | 布尔值 | 若为 `true`，则当此处理器执行失败时，Easysearch 将忽略该错误并继续执行搜索管道中的其余处理器。可选，默认值为 `false`。 |

## 示例

以下示例演示如何在搜索管道中使用 `filter_query` 处理器。

### 准备工作

创建一个名为 `my_index` 的索引，并索引两个文档：一个公开，一个私有：

```auto
POST /my_index/_doc/1
{
  "message": "This is a public message",
  "visibility": "public"
}
```

```auto
POST /my_index/_doc/2
{
  "message": "This is a private message",
  "visibility": "private"
}
```

### 创建搜索管道

以下请求创建一个名为 `my_pipeline` 的搜索管道，其中包含一个 `filter_query` 查询重写处理器。该处理器使用 term 查询，仅返回可见性为“public”的文档：

```auto
PUT /_search/pipeline/my_pipeline
{
  "rewrite_processors": [
    {
      "filter_query": {
        "tag": "tag1",
        "description": "此处理器将限制只返回可见性为公开的文档",
        "query": {
          "term": {
            "visibility": "public"
          }
        }
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

响应结果包含两个文档：

<details open markdown="block">
  <summary>
    响应
  </summary>

```auto
{
  "took": 47,
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
      },
      {
        "_index": "my_index",
        "_id": "2",
        "_score": 1.0,
        "_source": {
          "message": "This is a private message",
          "visibility": "private"
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

此时响应仅包含 `visibility` 为 `public` 的文档：

<details open markdown="block">
  <summary>
    响应
  </summary>

```auto
{
  "took": 19,
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
          "message": "This is a public message",
          "visibility": "public"
        }
      }
    ]
  }
}
```
</details>

---

