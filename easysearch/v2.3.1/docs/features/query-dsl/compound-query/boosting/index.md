---
title: "Boosting 查询"
date: 0001-01-01
summary: "Boosting 查询 #  如果你搜索&quot;pitcher&quot;这个词，你的结果可能既与棒球运动员有关，也与盛液体的容器有关。在棒球语境下搜索时，你可能想通过使用 must_not 子句完全排除包含&quot;glass&quot;或&quot;water&quot;的搜索结果。然而，如果你想保留这些结果但降低它们的关联度，可以使用 boosting 查询。
一个 boosting 查询返回与 positive 查询匹配的文档。在这些文档中，与 negative 查询也匹配的文档的关联度得分会降低（它们的关联度得分会乘以负的提升因子）。
相关指南（先读这些） #    查询 DSL 基础  相关性：加权与调参  参考用例 #  考虑一个包含两个文档的索引，你以如下方式索引：
PUT testindex/_doc/1 { &#34;article_name&#34;: &#34;The greatest pitcher in baseball history&#34; } PUT testindex/_doc/2 { &#34;article_name&#34;: &#34;The making of a glass pitcher&#34; } 使用以下匹配查询来搜索包含单词“pitcher”的文档：
GET testindex/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;article_name&#34;: &#34;pitcher&#34; } } } 返回的两个文档具有相同的相关性分数：
{ &#34;took&#34;: 5, &#34;timed_out&#34;: false, &#34;_shards&#34;: { &#34;total&#34;: 1, &#34;successful&#34;: 1, &#34;skipped&#34;: 0, &#34;failed&#34;: 0 }, &#34;hits&#34;: { &#34;total&#34;: { &#34;value&#34;: 2, &#34;relation&#34;: &#34;eq&#34; }, &#34;max_score&#34;: 0."
---


# Boosting 查询

如果你搜索"pitcher"这个词，你的结果可能既与棒球运动员有关，也与盛液体的容器有关。在棒球语境下搜索时，你可能想通过使用 `must_not` 子句完全排除包含"glass"或"water"的搜索结果。然而，如果你想保留这些结果但降低它们的关联度，可以使用 `boosting` 查询。

一个 `boosting` 查询返回与 `positive` 查询匹配的文档。在这些文档中，与 `negative` 查询也匹配的文档的关联度得分会降低（它们的关联度得分会乘以负的提升因子）。

## 相关指南（先读这些）

- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [相关性：加权与调参]({{< relref "/docs/features/fulltext-search/relevance/boosting.md" >}})

## 参考用例

考虑一个包含两个文档的索引，你以如下方式索引：

```
PUT testindex/_doc/1
{
  "article_name": "The greatest pitcher in baseball history"
}

PUT testindex/_doc/2
{
  "article_name": "The making of a glass pitcher"
}
```

使用以下匹配查询来搜索包含单词“pitcher”的文档：

```
GET testindex/_search
{
  "query": {
    "match": {
      "article_name": "pitcher"
    }
  }
}

```

返回的两个文档具有相同的相关性分数：

```
{
  "took": 5,
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
    "max_score": 0.18232156,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.18232156,
        "_source": {
          "article_name": "The greatest pitcher in baseball history"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.18232156,
        "_source": {
          "article_name": "The making of a glass pitcher"
        }
      }
    ]
  }
}

```

现在使用以下 `boosting` 查询来搜索包含单词“pitcher”的文档，但降低包含单词“glass”、“crystal”或“water”的文档的分数：

```
GET testindex/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "article_name": "pitcher"
        }
      },
      "negative": {
        "match": {
          "article_name": "glass crystal water"
        }
      },
      "negative_boost": 0.1
    }
  }
}

```

两个文档仍然会返回，但包含“glass”这个词的文档的相关性得分比之前的情况低 10 倍：

```
{
  "took": 13,
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
    "max_score": 0.18232156,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.18232156,
        "_source": {
          "article_name": "The greatest pitcher in baseball history"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.018232157,
        "_source": {
          "article_name": "The making of a glass pitcher"
        }
      }
    ]
  }
}

```

## 参数说明

下表列出了 `boosting` 查询支持的所有顶层参数。

| 参数             | 描述                                                                                                                               |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `positive`       | 文档必须匹配的查询，才能在结果中返回。必填。                                                                                       |
| `negative`       | 如果结果中的文档匹配此查询，其相关性得分会通过将其原始相关性得分（由 `positive` 查询产生）乘以 `negative_boost` 参数来降低。必填。 |
| `negative_boost` | 一个介于 0 和 1.0 之间的浮点数因子，用于将原始相关性分数乘以，以降低与 `negative` 查询匹配的文档的相关性。必填。                   |

