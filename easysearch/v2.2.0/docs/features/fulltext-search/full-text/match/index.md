---
title: "Match 查询"
date: 0001-01-01
summary: "Match 查询 #  使用 match 查询在特定文档字段上执行全文搜索。如果你在 text 字段上运行 match 查询，match 查询会分析提供的搜索字符串，并返回匹配字符串中任意词的文档。如果你在精确值字段上运行 match 查询，它会返回匹配精确值的文档。搜索精确值字段的推荐方式是使用过滤（filter）查询，因为与普通查询不同，过滤（filter）查询会被缓存。
相关指南（先读这些） #    全文检索  Query DSL 基础  以下示例展示了在 title 中对 wind 的基本 match 查询：
GET _search { &#34;query&#34;: { &#34;match&#34;: { &#34;title&#34;: &#34;wind&#34; } } } 通过传递其他参数，您可以使用扩展语法：
GET _search { &#34;query&#34;: { &#34;match&#34;: { &#34;title&#34;: { &#34;query&#34;: &#34;wind&#34;, &#34;analyzer&#34;: &#34;stop&#34; } } } } 参考样例 #  在以下示例中，您将使用包含以下文档的索引：
PUT testindex/_doc/1 { &#34;title&#34;: &#34;Let the wind rise&#34; } PUT testindex/_doc/2 { &#34;title&#34;: &#34;Gone with the wind&#34; } PUT testindex/_doc/3 { &#34;title&#34;: &#34;Rise is gone&#34; } 运算符（operator） #  operator 参数控制多个词元之间的逻辑关系：or（默认）或 and。默认操作符是 OR，查询 wind rise 会被改为 wind OR rise。要指定 and 操作符，请使用以下查询："
---


# Match 查询

使用 `match` 查询在特定文档字段上执行全文搜索。如果你在 `text` 字段上运行 `match` 查询，`match` 查询会分析提供的搜索字符串，并返回匹配字符串中任意词的文档。如果你在精确值字段上运行 `match` 查询，它会返回匹配精确值的文档。搜索精确值字段的推荐方式是使用过滤（`filter`）查询，因为与普通查询不同，过滤（`filter`）查询会被缓存。

## 相关指南（先读这些）

- [全文检索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

以下示例展示了在 `title` 中对 `wind` 的基本 `match` 查询：

```
GET _search
{
  "query": {
    "match": {
      "title": "wind"
    }
  }
}

```

通过传递其他参数，您可以使用扩展语法：

```
GET _search
{
  "query": {
    "match": {
      "title": {
        "query": "wind",
        "analyzer": "stop"
      }
    }
  }
}
```

## 参考样例

在以下示例中，您将使用包含以下文档的索引：

```
PUT testindex/_doc/1
{
  "title": "Let the wind rise"
}

PUT testindex/_doc/2
{
  "title": "Gone with the wind"

}

PUT testindex/_doc/3
{
  "title": "Rise is gone"
}

```

## 运算符（operator）

`operator` 参数控制多个词元之间的逻辑关系：`or`（默认）或 `and`。默认操作符是 `OR`，查询 `wind rise` 会被改为 `wind OR rise`。要指定 `and` 操作符，请使用以下查询：

```
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wind rise",
        "operator": "and"
      }
    }
  }
}

```

使用 `operator: "and"` 时，查询构建为 `wind AND rise`，返回结果：

```
{
  "took": 17,
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
    "max_score": 1.2667098,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1.2667098,
        "_source": {
          "title": "Let the wind rise"
        }
      }
    ]
  }
}
```

## 最小匹配（minimum_should_match）

`minimum_should_match` 参数控制文档必须匹配的最小词数。示例：

```
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wind rise",
        "operator": "or",
        "minimum_should_match": 2
      }
    }
  }
}
```

当 `minimum_should_match: 2` 且只有两个词时，效果等同于 `and` 运算符，返回结果：

```
{
  "took": 23,
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
    "max_score": 1.2667098,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1.2667098,
        "_source": {
          "title": "Let the wind rise"
        }
      }
    ]
  }
}
```

## 分词器（analyzer）

`analyzer` 参数指定用于分析查询文本的分词器。默认使用字段映射中指定的分词器（通常是 `standard`）。要使用不同的分词器，可在查询中指定。例如，使用 `english` 分词器：

```
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "the wind rises",
        "operator": "and",
        "analyzer": "english"
      }
    }
  }
}

```

`english` 分词器会移除停用词并执行词干提取，返回结果：

```
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
    "max_score": 1.2667098,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1.2667098,
        "_source": {
          "title": "Let the wind rise"
        }
      }
    ]
  }
}
```

## 空查询

在某些情况下，分词器可能会从查询中移除所有标记。例如， english 分词器会移除停用词，所以在查询 and OR or 中，所有标记都会被移除。要检查分词器的行为，你可以使用分析 API：

```
GET testindex/_analyze
{
  "analyzer" : "english",
  "text" : "and OR or"
}

```

正如预期，该查询没有产生任何标记：

```
{
  "tokens": []
}
```

你可以通过 `zero_terms_query` 参数指定空查询的行为。将 `zero_terms_query` 设置为 `all` 会返回索引中的所有文档，而将其设置为 `none` 则不会返回任何文档：

```
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "and OR or",
        "analyzer" : "english",
        "zero_terms_query": "all"
      }
    }
  }
}
```

## 模糊性查询

为了考虑拼写错误，您可以在查询中指定 `fuzziness` 作为以下任一选项：

- 一个整数，指定允许的最大 Damerau-Levenshtein 距离。
- `AUTO`
  - 0-2 个字符的字符串必须完全匹配。
  - 3–5 个字符的字符串允许 1 次编辑。
  - 超过 5 个字符的字符串允许 2 次编辑。

将 `fuzziness` 设置为默认的 `AUTO` 值在大多数情况下效果最佳：

```
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wnid",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

token `wnid` 匹配 `wind` ，查询返回文档 1 和 2：

```
{
  "took": 31,
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
    "max_score": 0.47501624,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.47501624,
        "_source": {
          "title": "Let the wind rise"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.47501624,
        "_source": {
          "title": "Gone with the wind"
        }
      }
    ]
  }
}
```

## 前缀长度

拼写错误很少出现在单词的开头。因此，您可以指定匹配前缀的最小长度，以便在结果中返回文档。例如，您可以将前面的查询更改为包含 `prefix_length` ：

```
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wnid",
        "fuzziness": "AUTO",
        "prefix_length": 2
      }
    }
  }
}

```

前面的查询没有返回结果。如果你将 `prefix_length` 改为 1，则会返回文档 1 和 2，因为词 `wnid` 的首字母没有拼写错误。

### 移位错误

在前面这个例子中，词 `wnid` 包含一个移位错误（ `in` 被改为了 `ni` ）。默认情况下，模糊匹配允许移位错误，但你可以通过将 `fuzzy_transpositions` 设置为 `false` 来禁止它们：

```
GET testindex/_search
{
  "query": {
    "match": {
      "title": {
        "query": "wnid",
        "fuzziness": "AUTO",
        "fuzzy_transpositions": false
      }
    }
  }
}

```

现在查询没有返回结果。

## 同义词

如果你使用 `synonym_graph` 过滤器并且 `auto_generate_synonyms_phrase_query` 设置为 `true` （默认值），Easysearch 会将查询解析为词项，然后将这些词项组合起来生成一个短语查询以匹配多词同义词。例如，如果你指定 `ba,batting average` 作为同义词并搜索 `ba` ，Easysearch 会搜索 `ba OR "batting average"` 。

要使用连词匹配多词同义词，将 `auto_generate_synonyms_phrase_query` 设置为 `false` ：

```
GET /testindex/_search
{
  "query": {
    "match": {
      "text": {
        "query": "good ba",
        "auto_generate_synonyms_phrase_query": false
      }
    }
  }
}

```

生成的查询是 `ba OR (batting AND average)` 。

## 参数说明

该查询将字段名称（ `<field>` ）作为顶级参数接受：

```
GET _search
{
  "query": {
    "match": {
      "<field>": {
        "query": "text to search for",
        ...
      }
    }
  }
}

```

`<field>` 接受以下参数。除 `query` 外，所有参数都是可选的。

| 参数                                  | 数据类型                                               | 描述                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `query`                               | String                                                 | 用于搜索的查询字符串。必填。                                                                                                                                                                                                                                                                                                                                                                                           |
| `auto_generate_synonyms_phrase_query` | Boolean                                                | 指定是否自动为多词同义词创建匹配短语查询。例如，如果你将 ba,batting average 指定为同义词并搜索 ba ，Easysearch 会搜索 ba OR "batting average" （如果此选项为 true ）或 ba OR (batting AND average) （如果此选项为 false ）。默认值为 true 。                                                                                                                                                                           |
| `analyzer`                            | String                                                 | 用于对查询字符串文本进行分词的分词器。默认值为为 default_field 指定的索引时分词器。如果未为 default_field 指定分词器，则 analyzer 为索引的默认分词器。有关 index.query.default_field 的更多信息，请参阅动态索引级索引设置。                                                                                                                                                                                            |
| `boost`                               | Float                                                  | 通过给定的乘数提升子句的权重。适用于在复合查询中对子句进行加权。[0, 1) 范围内的值会降低相关性，而大于 1 的值会增加相关性。默认值为 1 。                                                                                                                                                                                                                                                                                |
| `fuzziness`                           | String                                                 | 在确定一个词是否匹配一个值时，将其从一个词转换为另一个词所需的字符编辑次数（插入、删除、替换或转换）。例如， wined 和 wind 之间的距离是 1。有效值为非负整数或 AUTO 。默认值 AUTO 根据每个词的长度选择值，对于大多数用例是一个不错的选择。                                                                                                                                                                              |
| `fuzzy_rewrite`                       | String                                                 | 确定 Easysearch 如何重写查询。有效值为 `constant_score`、`scoring_boolean`、`constant_score_boolean`、`top_terms_N`、`top_terms_boost_N` 和 `top_terms_blended_freqs_N`。如果 `fuzziness` 参数不是 0，查询默认使用 `top_terms_blended_freqs_${max_expansions}` 重写方法。默认值为 `constant_score`。                                                                                                             |
| `fuzzy_transpositions`                | Boolean                                                | 将 fuzzy_transpositions 设置为 true （默认）会在 fuzziness 选项的插入、删除和替换操作中添加相邻字符的交换。例如，如果 fuzzy_transpositions 为真（交换“n”和“i”），则 wind 和 wnid 之间的距离为 1；如果为假（删除“n”，插入“n”），则距离为 2。如果 fuzzy_transpositions 为假， rewind 和 wnid 与 wind 的距离相同（2），尽管从更以人为中心的观点来看， wnid 是一个明显的拼写错误。对于大多数用例，默认值是一个不错的选择。 |
| `lenient`                             | Boolean                                                | 将 lenient 设置为 true 会忽略查询与文档字段之间的数据类型不匹配。例如， "8.2" 的查询字符串可以匹配类型为 float 的字段。默认值为 false 。                                                                                                                                                                                                                                                                               |
| `max_expansions`                      | Positive integer                                       | 查询可以扩展到的最大词数。模糊查询会扩展到与指定距离（`fuzziness`）内的匹配词。然后 Easysearch 尝试匹配这些词。默认值为 50。                                                                                                                                                                                                                                                                                          |
| `minimum_should_match`                | 正整数或负整数、正百分比或负百分比、也可以是这些的组合 | 如果查询字符串包含多个搜索词并且你使用 or 运算符，文档被考虑为匹配所需的匹配词数。例如，如果 minimum_should_match 为 2， wind often rising 不匹配 The Wind Rises. 如果 minimum_should_match 为 1 ，则匹配。详情请参阅 Minimum should match。                                                                                                                                                                           |
| `operator`                            | String                                                 | 如果查询字符串包含多个搜索词，文档是否需要所有词都匹配（ AND ）或只需一个词匹配（ OR ）才被视为匹配。有效值为：- OR : 字符串 to be 被解释为 to OR be ；- AND : 字符串 to be 被解释为 to AND be 。默认值为 OR 。                                                                                                                                                                                                        |
| `prefix_length`                       | Non-negative integer                                   | 不考虑模糊性的前导字符数量。默认为 0 。                                                                                                                                                                                                                                                                                                                                                                                |
| `zero_terms_query`                    | String                                                 | 在某些情况下，分词器会从查询字符串中删除所有词项。例如， stop 分词器会从字符串 an but this 中删除所有词项。在这种情况下， zero_terms_query 指定是否匹配不匹配任何文档（ none ）或所有文档（ all ）。有效值为 none 和 all 。默认为 none 。                                                                                                                                                                              |

