---
title: "Dis Max 查询"
date: 0001-01-01
summary: "Dis Max 查询 #  dis_max 查询返回与一个或多个查询子句匹配的任何文档。对于与多个查询子句匹配的文档，相关性得分设置为所有匹配查询子句中的最高相关性得分。
当返回的文档的相关性分数相同时，您可以使用 tie_breaker 参数来增加匹配多个查询子句的文档的权重。
相关指南（先读这些） #    多字段搜索  Query DSL 基础  参考样例 #  考虑一个包含两个文档的索引，您按照以下方式索引这些文档：
PUT testindex1/_doc/1 { &#34;title&#34;: &#34; The Top 10 Shakespeare Poems&#34;, &#34;description&#34;: &#34;Top 10 sonnets of England&#39;s national poet and the Bard of Avon&#34; } PUT testindex1/_doc/2 { &#34;title&#34;: &#34;Sonnets of the 16th Century&#34;, &#34;body&#34;: &#34;The poems written by various 16-th century poets&#34; } 使用 dis_max 查询来搜索包含单词“莎士比亚诗歌”的文档"
---


# Dis Max 查询

`dis_max` 查询返回与一个或多个查询子句匹配的任何文档。对于与多个查询子句匹配的文档，相关性得分设置为所有匹配查询子句中的最高相关性得分。

当返回的文档的相关性分数相同时，您可以使用 `tie_breaker` 参数来增加匹配多个查询子句的文档的权重。

## 相关指南（先读这些）

- [多字段搜索]({{< relref "/docs/features/fulltext-search/multi-field-search.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

考虑一个包含两个文档的索引，您按照以下方式索引这些文档：

```
PUT testindex1/_doc/1
{
  "title": " The Top 10 Shakespeare Poems",
  "description": "Top 10 sonnets of England's national poet and the Bard of Avon"
}

PUT testindex1/_doc/2
{
  "title": "Sonnets of the 16th Century",
  "body": "The poems written by various 16-th century poets"
}
```

使用 `dis_max` 查询来搜索包含单词“莎士比亚诗歌”的文档

```
GET testindex1/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "title": "Shakespeare poems" }},
        { "match": { "body":  "Shakespeare poems" }}
      ]
    }
  }
}
```

返回内容包含两个文档：

```
{
  "took": 8,
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
    "max_score": 1.3862942,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "1",
        "_score": 1.3862942,
        "_source": {
          "title": " The Top 10 Shakespeare Poems",
          "description": "Top 10 sonnets of England's national poet and the Bard of Avon"
        }
      },
      {
        "_index": "testindex1",
        "_id": "2",
        "_score": 0.2876821,
        "_source": {
          "title": "Sonnets of the 16th Century",
          "body": "The poems written by various 16-th century poets"
        }
      }
    ]
  }
}

```

## 参数说明

以下表格列出了所有由 `dis_max` 查询支持的一级参数。

| 参数          | 描述                                                                                                                                                                                                                                                                                            |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `queries`     | 一个或多个查询子句的数组，用于匹配文档。文档必须至少匹配一个查询子句才能在结果中返回。如果文档匹配多个查询子句，则相关度分数设置为所有匹配查询子句中的最高相关度分数。必需的。                                                                                                                  |
| `tie_breaker` | 一个介于 0 和 1.0 之间的浮点因子，用于给匹配多个查询子句的文档赋予更多权重。在这种情况下，文档的相关性得分使用以下算法计算：从所有匹配的查询子句中取最高相关性得分，将所有其他匹配子句的得分乘以 `tie_breaker` 值，然后将相关性得分相加，并进行归一化。可选。默认值为 0（表示只计算最高得分）。 |

