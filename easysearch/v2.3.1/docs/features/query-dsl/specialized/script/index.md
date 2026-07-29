---
title: "Script 查询"
date: 0001-01-01
summary: "Script 查询 #  使用 script 查询基于 Painless 脚本语言编写的自定义条件来过滤文档。此查询返回脚本评估结果为 true 的文档，从而实现无法使用标准查询表达的高级过滤逻辑。
相关指南（先读这些） #    Query DSL 基础  结构化搜索  专业查询（Specialized queries）   性能注意：script 查询计算成本高，应谨慎使用。仅在必要时使用，并确保 search.allow_expensive_queries 已启用（默认为 true ）。有关更多信息，请参阅昂贵查询。
 参考样例 #  使用以下映射创建一个名为 products 的索引：
PUT /products { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;price&#34;: { &#34;type&#34;: &#34;float&#34; }, &#34;rating&#34;: { &#34;type&#34;: &#34;float&#34; } } } } 使用以下请求索引示例文档：
POST /products/_bulk { &#34;index&#34;: { &#34;_id&#34;: 1 } } { &#34;title&#34;: &#34;Wireless Earbuds&#34;, &#34;price&#34;: 99."
---


# Script 查询

使用 `script` 查询基于 `Painless` 脚本语言编写的自定义条件来过滤文档。此查询返回脚本评估结果为 `true` 的文档，从而实现无法使用标准查询表达的高级过滤逻辑。

## 相关指南（先读这些）

- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})
- [专业查询（Specialized queries）]({{< relref "/docs/features/query-dsl/specialized/_index.md" >}})

> **性能注意**：script 查询计算成本高，应谨慎使用。仅在必要时使用，并确保 search.allow_expensive_queries 已启用（默认为 true ）。有关更多信息，请参阅昂贵查询。

## 参考样例

使用以下映射创建一个名为 `products` 的索引：

```
PUT /products
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "price": { "type": "float" },
      "rating": { "type": "float" }
    }
  }
}
```

使用以下请求索引示例文档：

```
POST /products/_bulk
{ "index": { "_id": 1 } }
{ "title": "Wireless Earbuds", "price": 99.99, "rating": 4.5 }
{ "index": { "_id": 2 } }
{ "title": "Bluetooth Speaker", "price": 79.99, "rating": 4.8 }
{ "index": { "_id": 3 } }
{ "title": "Noise Cancelling Headphones", "price": 199.99, "rating": 4.7 }
```

## 基本脚本查询

返回评分高于 4.6 的产品：

```
POST /products/_search
{
  "query": {
    "script": {
      "script": {
        "source": "doc['rating'].value > 4.6"
      }
    }
  }
}
```

返回的命中结果仅包括 `rating` 值高于 4.6 的文档：

```
{
  ...
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "products",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "Bluetooth Speaker",
          "price": 79.99,
          "rating": 4.8
        }
      },
      {
        "_index": "products",
        "_id": "3",
        "_score": 1,
        "_source": {
          "title": "Noise Cancelling Headphones",
          "price": 199.99,
          "rating": 4.7
        }
      }
    ]
  }
}

```

## 参数说明

`script` 查询采用以下顶级参数。

| 参数            | 必需/可选 | 描述                                      |
| --------------- | --------- | ----------------------------------------- |
| `script.source` | 必需      | 计算结果为 `true` 或 `false` 的脚本代码。 |
| `script.params` | 可选      | 脚本内部引用的用户定义参数。              |

## 使用脚本参数

你可以使用 `params` 来安全地注入值，利用脚本编译缓存：

```
POST /products/_search
{
  "query": {
    "script": {
      "script": {
        "source": "doc['price'].value < params.max_price",
        "params": {
          "max_price": 100
        }
      }
    }
  }
}
```

返回的命中结果仅包括文档的 `price` 值小于 100 的文档：

```
{
  ...
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "products",
        "_id": "1",
        "_score": 1,
        "_source": {
          "title": "Wireless Earbuds",
          "price": 99.99,
          "rating": 4.5
        }
      },
      {
        "_index": "products",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "Bluetooth Speaker",
          "price": 79.99,
          "rating": 4.8
        }
      }
    ]
  }
}

```

## 组合多个条件

使用以下查询来搜索产品，其 `rating` 高于 4.5 且 `price` 低于 100 :

```
POST /products/_search
{
  "query": {
    "script": {
      "script": {
        "source": "doc['rating'].value > 4.5 && doc['price'].value < 100"
      }
    }
  }
}
```

仅返回符合要求的文档：

```
{
  "took": 12,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "products",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "Bluetooth Speaker",
          "price": 79.99,
          "rating": 4.8
        }
      }
    ]
  }
}
```

