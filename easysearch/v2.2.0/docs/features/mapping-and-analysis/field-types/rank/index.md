---
title: "排名特征字段类型（Rank Feature）"
date: 0001-01-01
summary: "Rank 字段类型 #  下表列出了 Easysearch 支持的所有 rank 字段类型。
   字段数据类型 描述     rank_feature 提升或降低文档的相关性得分。   rank_features 提升或降低文档的相关性得分。用于特征列表稀疏的情况。     注意：rank_feature 和 rank_features 字段只能使用 rank_feature 查询进行查询。它们不支持聚合或排序。
 相关指南（先读这些） #    映射基础  相关性：加权与调参  排序功能查询  Rank feature 字段类型 #  Rank feature 字段类型使用正浮点值来提升或降低文档在 rank_feature 查询中的相关性得分。默认情况下，该值会提升相关性得分。要降低相关性得分，请将可选参数 positive_score_impact 设置为 false。
示例 #  创建一个包含 rank feature 字段的映射：
PUT chessplayers { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;name&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;rating&#34;: { &#34;type&#34;: &#34;rank_feature&#34; }, &#34;age&#34;: { &#34;type&#34;: &#34;rank_feature&#34;, &#34;positive_score_impact&#34;: false } } } } 索引三个文档，其中包含一个提升得分的 rank_feature 字段（rating）和一个降低得分的 rank_feature 字段（age）："
---


# Rank 字段类型

下表列出了 Easysearch 支持的所有 rank 字段类型。

| 字段数据类型    | 描述                                                 |
| --------------- | ---------------------------------------------------- |
| `rank_feature`  | 提升或降低文档的相关性得分。                         |
| `rank_features` | 提升或降低文档的相关性得分。用于特征列表稀疏的情况。 |

> **注意**：`rank_feature` 和 `rank_features` 字段只能使用 `rank_feature` 查询进行查询。它们不支持聚合或排序。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [相关性：加权与调参]({{< relref "/docs/features/fulltext-search/relevance/boosting.md" >}})
- [排序功能查询]({{< relref "/docs/features/query-dsl/specialized/rank-feature.md" >}})

## Rank feature 字段类型

Rank feature 字段类型使用正浮点值来提升或降低文档在 `rank_feature` 查询中的相关性得分。默认情况下，该值会提升相关性得分。要降低相关性得分，请将可选参数 `positive_score_impact` 设置为 false。

### 示例

创建一个包含 rank feature 字段的映射：

```json
PUT chessplayers
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "rating": {
        "type": "rank_feature"
      },
      "age": {
        "type": "rank_feature",
        "positive_score_impact": false
      }
    }
  }
}
```

索引三个文档，其中包含一个提升得分的 rank_feature 字段（`rating`）和一个降低得分的 rank_feature 字段（`age`）：

```json
PUT chessplayers/_doc/1
{
  "name": "John Doe",
  "rating": 2554,
  "age": 75
}

PUT chessplayers/_doc/2
{
  "name": "Kwaku Mensah",
  "rating": 2067,
  "age": 10
}

PUT chessplayers/_doc/3
{
  "name": "Nikki Wolf",
  "rating": 1864,
  "age": 22
}
```

## Rank feature 查询

使用 rank feature 查询，您可以按评分、年龄或同时按评分和年龄对选手进行排名。如果按评分排名，评分较高的选手将获得更高的相关性得分。如果按年龄排名，年龄较小的选手将获得更高的相关性得分。

使用 rank feature 查询根据年龄和评分搜索选手：

```json
GET chessplayers/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "rank_feature": {
            "field": "rating"
          }
        },
        {
          "rank_feature": {
            "field": "age"
          }
        }
      ]
    }
  }
}
```

当同时按年龄和评分排名时，年龄较小和评分较高的选手得分更好：

```json
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
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1.2093145,
    "hits": [
      {
        "_index": "chessplayers",
        "_type": "_doc",
        "_id": "2",
        "_score": 1.2093145,
        "_source": {
          "name": "Kwaku Mensah",
          "rating": 1967,
          "age": 10
        }
      },
      {
        "_index": "chessplayers",
        "_type": "_doc",
        "_id": "3",
        "_score": 1.0150313,
        "_source": {
          "name": "Nikki Wolf",
          "rating": 1864,
          "age": 22
        }
      },
      {
        "_index": "chessplayers",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.8098284,
        "_source": {
          "name": "John Doe",
          "rating": 2554,
          "age": 75
        }
      }
    ]
  }
}
```

## Rank features 字段类型

Rank features 字段类型与 rank feature 字段类型类似，但它更适合用于稀疏特征列表。Rank features 字段可以索引数值特征向量，这些向量后续用于在 `rank_feature` 查询中提升或降低文档的相关性得分。

### 示例

创建一个包含 rank features 字段的映射：

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "correlations": {
        "type": "rank_features"
      }
    }
  }
}
```

要索引包含 rank features 字段的文档，请使用带有字符串键和正浮点值的哈希映射：

```json
PUT testindex1/_doc/1
{
  "correlations": {
    "young kids": 1,
    "older kids": 15,
    "teens": 25.9
  }
}

PUT testindex1/_doc/2
{
  "correlations": {
    "teens": 10,
    "adults": 95.7
  }
}
```

使用 rank feature 查询检索文档：

```json
GET testindex1/_search
{
  "query": {
    "rank_feature": {
      "field": "correlations.teens"
    }
  }
}
```

响应按相关性得分排序：

```json
{
  "took": 123,
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
    "max_score": 0.6258503,
    "hits": [
      {
        "_index": "testindex1",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.6258503,
        "_source": {
          "correlations": {
            "young kids": 1,
            "older kids": 15,
            "teens": 25.9
          }
        }
      },
      {
        "_index": "testindex1",
        "_type": "_doc",
        "_id": "2",
        "_score": 0.39263803,
        "_source": {
          "correlations": {
            "teens": 10,
            "adults": 95.7
          }
        }
      }
    ]
  }
}
```

> Rank feature 和 rank features 字段使用前 9 个有效位来保证精度，导致大约 0.4% 的相对误差。值的存储相对精度为 2^−8 = 0.00390625。

