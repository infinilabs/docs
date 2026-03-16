---
title: "Bool 查询"
date: 0001-01-01
summary: "Bool 查询 #  bool 查询可以将多个查询子句组合成一个高级查询。这些子句通过布尔逻辑组合起来，以在结果中找到匹配的文档。
相关指南（先读这些） #    Query DSL 基础  结构化搜索  查询子句 #  在布尔（ bool ）查询中使用以下查询子句：
   子句 行为     must 逻辑 and 运算符。结果必须匹配此子句中的所有查询。   must_not 逻辑 not 运算符。所有匹配项都将被排除在结果之外。如果 must_not 包含多个子句，则只返回不匹配任何这些子句的文档。例如， &quot;must_not&quot;:[{clause_A}, {clause_B}] 等同于 NOT(A OR B) 。   should 逻辑 or 运算符。结果必须匹配至少一个查询。匹配更多 should 子句会增加文档的相关性分数。您可以使用 minimum_should_match 参数设置必须匹配的最小查询数量。如果一个查询包含 must 或 filter 子句，默认 minimum_should_match 值为 0。否则，默认 minimum_should_match 值为 1。   filter 逻辑 and 运算符，在应用查询之前首先应用于减少您的数据集。过滤器子句中的查询是一个是或否选项。如果文档匹配查询，则它将出现在结果中；否则，它将不会出现。过滤器查询的结果通常会被缓存以允许更快的返回。使用过滤器查询根据精确匹配、范围、日期或数字来过滤结果。    一个布尔查询具有以下结构："
---


# Bool 查询

`bool` 查询可以将多个查询子句组合成一个高级查询。这些子句通过布尔逻辑组合起来，以在结果中找到匹配的文档。

## 相关指南（先读这些）

- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

## 查询子句

在布尔（ `bool` ）查询中使用以下查询子句：

| 子句       | 行为                                                                                                                                                                                                                                                                                 |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `must`     | 逻辑 `and` 运算符。结果必须匹配此子句中的所有查询。                                                                                                                                                                                                                                  |
| `must_not` | 逻辑 `not` 运算符。所有匹配项都将被排除在结果之外。如果 `must_not` 包含多个子句，则只返回不匹配任何这些子句的文档。例如， `"must_not":[{clause_A}, {clause_B}]` 等同于 `NOT(A OR B)` 。                                                                                              |
| `should`   | 逻辑 `or` 运算符。结果必须匹配至少一个查询。匹配更多 `should` 子句会增加文档的相关性分数。您可以使用 `minimum_should_match` 参数设置必须匹配的最小查询数量。如果一个查询包含 `must` 或 `filter` 子句，默认 `minimum_should_match` 值为 0。否则，默认 `minimum_should_match` 值为 1。 |
| `filter`   | 逻辑 `and` 运算符，在应用查询之前首先应用于减少您的数据集。过滤器子句中的查询是一个是或否选项。如果文档匹配查询，则它将出现在结果中；否则，它将不会出现。过滤器查询的结果通常会被缓存以允许更快的返回。使用过滤器查询根据精确匹配、范围、日期或数字来过滤结果。                      |

一个布尔查询具有以下结构：

```
GET _search
{
  "query": {
    "bool": {
      "must": [
        {}
      ],
      "must_not": [
        {}
      ],
      "should": [
        {}
      ],
      "filter": {}
    }
  }
}

```

例如，假设您在 Easysearch 集群中索引了莎士比亚的全部作品。您希望构建一个满足以下要求的单个查询：

1. `text_entry` 字段必须包含单词 `love` ，并且应该包含 `life` 或 `grace` 。
2. `speaker` 字段不能包含 `ROMEO` 。
3. 将结果过滤到排位 `Romeo and Juliet` ，而不影响相关性分数。

这些要求可以在以下查询中组合：

```
GET shakespeare/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text_entry": "love"
          }
        }
      ],
      "should": [
        {
          "match": {
            "text_entry": "life"
          }
        },
        {
          "match": {
            "text_entry": "grace"
          }
        }
      ],
      "minimum_should_match": 1,
      "must_not": [
        {
          "match": {
            "speaker": "ROMEO"
          }
        }
      ],
      "filter": {
        "term": {
          "play_name": "Romeo and Juliet"
        }
      }
    }
  }
}
```

返回内容包含匹配文档：

```
{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 4,
    "successful": 4,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 11.356054,
    "hits": [
      {
        "_index": "shakespeare",
        "_id": "88020",
        "_score": 11.356054,
        "_source": {
          "type": "line",
          "line_id": 88021,
          "play_name": "Romeo and Juliet",
          "speech_number": 19,
          "line_number": "4.5.61",
          "speaker": "PARIS",
          "text_entry": "O love! O life! not life, but love in death!"
        }
      }
    ]
  }
}

```

如果你想要识别是这些子句中的哪一个实际导致了匹配结果，请使用 `_name` 参数为每个查询命名。要添加 `_name` 参数，请将 `match` 查询中的字段名更改为对象：

```
GET shakespeare/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "text_entry": {
              "query": "love",
              "_name": "love-must"
            }
          }
        }
      ],
      "should": [
        {
          "match": {
            "text_entry": {
              "query": "life",
              "_name": "life-should"
            }
          }
        },
        {
          "match": {
            "text_entry": {
              "query": "grace",
              "_name": "grace-should"
            }
          }
        }
      ],
      "minimum_should_match": 1,
      "must_not": [
        {
          "match": {
            "speaker": {
              "query": "ROMEO",
              "_name": "ROMEO-must-not"
            }
          }
        }
      ],
      "filter": {
        "term": {
          "play_name": "Romeo and Juliet"
        }
      }
    }
  }
}
```

Easysearch 返回一个 `matched_queries` 数组，列出了匹配这些结果的查询：

```
"matched_queries": [
  "love-must",
  "life-should"
]
```

如果你移除不在该列表中的查询，你仍然会看到完全相同的结果。通过检查哪个 `should` 子句匹配了，你可以更好地理解结果的相关性分数。

您还可以通过嵌套 `bool` 查询来构建复杂的布尔表达式。例如，使用以下查询在 `play Romeo and Juliet` 中查找匹配 `( love OR hate ) AND ( life OR grace )` 的 `text_entry` 字段：

```
GET shakespeare/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "bool": {
            "should": [
              {
                "match": {
                  "text_entry": "love"
                }
              },
              {
                "match": {
                  "text": "hate"
                }
              }
            ]
          }
        },
        {
          "bool": {
            "should": [
              {
                "match": {
                  "text_entry": "life"
                }
              },
              {
                "match": {
                  "text": "grace"
                }
              }
            ]
          }
        }
      ],
      "filter": {
        "term": {
          "play_name": "Romeo and Juliet"
        }
      }
    }
  }
}

```

返回内容包含匹配文档：

```
{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 2,
    "successful": 2,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 11.37006,
    "hits": [
      {
        "_index": "shakespeare",
        "_type": "doc",
        "_id": "88020",
        "_score": 11.37006,
        "_source": {
          "type": "line",
          "line_id": 88021,
          "play_name": "Romeo and Juliet",
          "speech_number": 19,
          "line_number": "4.5.61",
          "speaker": "PARIS",
          "text_entry": "O love! O life! not life, but love in death!"
        }
      }
    ]
  }
}
```

## 完整参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| `must` | Array of Query | 文档必须匹配的查询。参与评分。 |
| `filter` | Array of Query | 文档必须匹配的查询。不参与评分，结果可缓存。 |
| `should` | Array of Query | 文档应该匹配的查询。增加 `_score`。 |
| `must_not` | Array of Query | 文档必须不匹配的查询。不参与评分。 |
| `minimum_should_match` | Integer/String | should 子句的最少匹配数。详见 [minimum_should_match]({{< relref "/docs/features/query-dsl/minimum-should-match.md" >}})。 |
| `boost` | Float | 整个 bool 查询的权重。默认 1.0。 |
| `adjust_pure_negative` | Boolean | 当 bool 查询只包含 `must_not`（纯否定查询）时，是否自动添加 `match_all` 使查询正常工作。默认 `true`。通常不需要修改。 |
| `_name` | String | 查询标签名称，用于 `matched_queries` 响应。 |

