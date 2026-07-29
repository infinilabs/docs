---
title: "Multi Match 查询"
date: 0001-01-01
summary: "Multi Match 查询 #  multi_match 查询与 match 查询类似。您可以使用 multi_match 查询来搜索多个字段。
相关指南（先读这些） #    多字段搜索  全文搜索  字段权重 #  ^ 会&quot;提升&quot;某些字段的权重。提升是乘数，用于使一个字段中的匹配比其他字段中的匹配更重要。在以下示例中，title字段中匹配 &ldquo;wind&rdquo; 的权重比 plot 字段中匹配的权重高 _score 四倍：
GET _search { &#34;query&#34;: { &#34;multi_match&#34;: { &#34;query&#34;: &#34;wind&#34;, &#34;fields&#34;: [&#34;title^4&#34;, &#34;plot&#34;] } } } 结果是，像《The Wind Rises》和《Gone with the Wind》这样的电影出现在搜索结果的顶部附近，而像《Twister》这样的电影，其剧情简介中可能包含“wind”字，则出现在底部附近。
您可以在字段名中使用通配符。例如，以下查询将搜索 speaker 字段以及所有以 play_ 开头的字段，例如 play_name 或 play_title ：
GET _search { &#34;query&#34;: { &#34;multi_match&#34;: { &#34;query&#34;: &#34;hamlet&#34;, &#34;fields&#34;: [&#34;speaker&#34;, &#34;play_*&#34;] } } } 如果您不提供 fields 参数，multi_match 查询将搜索 index."
---


# Multi Match 查询

`multi_match` 查询与 `match` 查询类似。您可以使用 `multi_match` 查询来搜索多个字段。

## 相关指南（先读这些）

- [多字段搜索]({{< relref "/docs/features/fulltext-search/multi-field-search.md" >}})
- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})

## 字段权重

`^` 会"提升"某些字段的权重。提升是乘数，用于使一个字段中的匹配比其他字段中的匹配更重要。在以下示例中，`title`字段中匹配 "wind" 的权重比 `plot` 字段中匹配的权重高 `_score` 四倍：

```
GET _search
{
  "query": {
    "multi_match": {
      "query": "wind",
      "fields": ["title^4", "plot"]
    }
  }
}

```

结果是，像`《The Wind Rises》`和`《Gone with the Wind》`这样的电影出现在搜索结果的顶部附近，而像`《Twister》`这样的电影，其剧情简介中可能包含“wind”字，则出现在底部附近。

您可以在字段名中使用通配符。例如，以下查询将搜索 `speaker` 字段以及所有以 `play_` 开头的字段，例如 `play_name` 或 `play_title` ：

```
GET _search
{
  "query": {
    "multi_match": {
      "query": "hamlet",
      "fields": ["speaker", "play_*"]
    }
  }
}
```

如果您不提供 `fields` 参数，`multi_match` 查询将搜索 `index.query.default_field` 设置中指定的字段，该设置默认为 `*`。默认行为是提取映射中所有适用于词级查询的字段，过滤元数据字段，并将所有提取的字段组合起来构建查询。

> 查询中的子句最大数量由 indices.query.bool.max_clause_count 设置定义，默认为 1,024。

## 多字段查询类型

Easysearch 支持以下多字段查询类型，它们在内部执行查询的方式上有所不同：

- `best_fields` （默认）：返回匹配任何字段的文档。使用最佳匹配字段的 `_score` 。
- `most_fields` ：返回匹配任何字段的文档。使用每个匹配字段的组合得分。
- `cross_fields` : 将所有字段视为一个字段。处理具有相同 `analyzer` 的字段，并在任何字段中匹配词语。
- `phrase` : 在每个字段上运行 `match_phrase` 查询。使用最佳匹配字段的 `_score` 。
- `phrase_prefix` : 在每个字段上运行 `match_phrase_prefix` 查询。使用最佳匹配字段的 `_score` 。
- `bool_prefix` : 在每个字段上运行 `match_bool_prefix` 查询。使用每个匹配字段的组合得分。

## Best fields 最佳字段

如果你在搜索两个指定概念的字词，你希望包含这两个字词相邻的结果得分更高。

例如，创建一个包含以下科学文章的索引：

```
PUT /articles/_doc/1
{
  "title": "Aurora borealis",
  "description": "Northern lights, or aurora borealis, explained"
}

PUT /articles/_doc/2
{
  "title": "Sun deprivation in the Northern countries",
  "description": "Using fluorescent lights for therapy"
}
```

你可以搜索标题或描述中包含 `northern lights` 的文章：

```
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "northern lights",
      "type": "best_fields",
      "fields": [ "title", "description" ],
      "tie_breaker": 0.3
    }
  }
}
```

前面的查询被执行为以下 `dis_max` 查询，每个字段都有一个 `match` 查询：

```
GET /articles/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "title": "northern lights" }},
        { "match": { "description": "northern lights" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
```

结果包含这两个文档，但文档 1 的得分更高，因为这两个词都在 `description` 字段中：

```
{
  "took": 30,
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
    "max_score": 0.84407747,
    "hits": [
      {
        "_index": "articles",
        "_id": "1",
        "_score": 0.84407747,
        "_source": {
          "title": "Aurora borealis",
          "description": "Northern lights, or aurora borealis, explained"
        }
      },
      {
        "_index": "articles",
        "_id": "2",
        "_score": 0.6322521,
        "_source": {
          "title": "Sun deprivation in the Northern countries",
          "description": "Using fluorescent lights for therapy"
        }
      }
    ]
  }
}

```

`best_fields` 查询使用最佳匹配字段的分数。如果你指定了 `tie_breaker` ，分数将使用以下算法计算：

取最佳匹配字段的得分，并为所有其他匹配字段加上（ `tie_breaker * _score` ）。

## Most fields 大多数字段

使用 `most_fields` 查询针对包含相同文本但分析方式不同的多个字段。例如，原始字段可能包含使用 `standard` 分词器分析的文本，而另一个字段可能包含使用执行词干提取的 `english` 分词器分析的相同文本：

```
PUT /articles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "english": {
            "type": "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}

```

考虑以下两个索引在 `articles` 中的文档：

```
PUT /articles/_doc/1
{
  "title": "Buttered toasts"
}

PUT /articles/_doc/2
{
  "title": "Buttering a toast"
}

```

`standard` 分词器将标题 `Buttered toast` 分析为 [ `buttered` , `toasts` ]，并将标题 `Buttering a toast` 分析为 [ `buttering` , `a` , `toast` ]。另一方面，由于词干提取， `english` 分词器对两个标题都产生相同的词列表 [ `butter` , `toast` ]。

您可以使用 `most_fields` 查询以返回尽可能多的文档：

```
GET /articles/_search
{
  "query": {
    "multi_match": {
      "query": "buttered toast",
      "fields": [
        "title",
        "title.english"
      ],
      "type": "most_fields"
    }
  }
}
```

前面的查询作为以下布尔查询执行：

```
GET articles/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "buttered toasts" }},
        { "match": { "title.english": "buttered toasts" }}
      ]
    }
  }
}
```

要计算相关性分数，文档中所有 `match` 子句的分数会被加在一起，然后结果再除以 `match` 子句的数量。

包含 `title.english` 字段会检索到与词干化标记匹配的第二份文档：

```
{
  "took": 9,
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
    "max_score": 1.4418206,
    "hits": [
      {
        "_index": "articles",
        "_id": "1",
        "_score": 1.4418206,
        "_source": {
          "title": "Buttered toasts"
        }
      },
      {
        "_index": "articles",
        "_id": "2",
        "_score": 0.09304003,
        "_source": {
          "title": "Buttering a toast"
        }
      }
    ]
  }
}

```

由于第一个文档的 `title` 和 `title.english` 字段都匹配，因此它的相关性得分更高。

## 操作符 Operator 和 最小值应匹配 minimum_should_match

`best_fields` 和 `most_fields` 查询基于字段生成匹配查询（每个字段一个）。因此， `minimum_should_match` 和 `operator` 参数应用于每个字段，这通常不是期望的行为。

例如，创建一个 `customers` 索引，其中包含以下文档：

```
PUT customers/_doc/1
{
  "first_name": "John",
  "last_name": "Doe"
}

PUT customers/_doc/2
{
  "first_name": "Jane",
  "last_name": "Doe"
}
```

如果你在 `customers` 索引中搜索 `John Doe` ，你可能会构建以下查询：

```
GET customers/_validate/query?explain
{
  "query": {
    "multi_match" : {
      "query": "John Doe",
      "type": "best_fields",
      "fields": [ "first_name", "last_name" ],
      "operator": "and"
    }
  }
}

```

`and` 运算符在此查询中的意图是找到匹配 `John` 和 `Doe` 的文档。然而，查询没有返回任何结果。您可以通过运行验证 `API` 来学习查询是如何执行的：

```
GET customers/_validate/query?explain
{
  "query": {
    "multi_match" : {
      "query":      "John Doe",
      "type":       "best_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and"
    }
  }
}
```

从返回内容中可以看出，查询试图将 `John` 和 `Doe` 匹配到 `first_name` 或 `last_name` 字段：

```
{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "valid": true,
  "explanations": [
    {
      "index": "customers",
      "valid": true,
      "explanation": "((+first_name:john +first_name:doe) | (+last_name:john +last_name:doe))"
    }
  ]
}
```

因为两个字段都不包含这两个词，所以没有返回结果。

跨字段搜索的更好替代方案是使用 `cross_fields` 查询。与以字段为中心的 `best_fields` 和 `most_fields` 查询不同， `cross_fields` 查询是以词为中心的。

## 跨字段

使用 `cross_fields` 查询来跨多个字段搜索数据。例如，如果索引包含客户数据，客户的名和姓位于不同的字段中。但是，当你搜索 `John Doe` 时，你希望获得文档，其中 `John` 在 `first_name` 字段中， `Doe` 在 `last_name` 字段中。

`most_fields` 查询在这种情况下无法工作，因为存在以下问题：

- `operator` 和 `minimum_should_match` 参数是按字段应用，而不是按词应用。
- `first_name` 和 `last_name` 字段的词频可能导致意外结果。例如，如果某人的`first_name`恰好是 `Doe` ，包含这个名字的文档会被认为是一个更好的匹配，因为这个名字不会出现在其他任何文档中。

`cross_fields` 查询会将查询字符串分析为单独的词，然后在任何字段中搜索每个词，就像它们是一个字段一样。

以下是 `cross_fields` 查询 `John Doe` 的内容：

```
GET /customers/_search
{
  "query": {
    "multi_match" : {
      "query": "John Doe",
      "type": "cross_fields",
      "fields": [ "first_name", "last_name" ],
      "operator": "and"
    }
  }
}

```

返回包含唯一一份文档，其中同时包含 `John` 和 `Doe` ：

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
    "max_score": 0.8754687,
    "hits": [
      {
        "_index": "customers",
        "_id": "1",
        "_score": 0.8754687,
        "_source": {
          "first_name": "John",
          "last_name": "Doe"
        }
      }
    ]
  }
}

```

您可以使用验证 API 操作来了解前面的查询是如何执行的：

```
GET /customers/_validate/query?explain
{
  "query": {
    "multi_match" : {
      "query": "John Doe",
      "type": "cross_fields",
      "fields": [ "first_name", "last_name" ],
      "operator": "and"
    }
  }
}
```

从返回内容中可以看出，查询是在至少一个字段中搜索所有词项：

```
{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "valid": true,
  "explanations": [
    {
      "index": "customers",
      "valid": true,
      "explanation": "+blended(terms:[last_name:john, first_name:john]) +blended(terms:[last_name:doe, first_name:doe])"
    }
  ]
}
```

因此，通过混合所有字段的词频，解决了因词频差异而造成的问题，从而纠正了这些差异。

> `cross_fields` 查询通常只在各字段的 `boost` 值为 1 的情况下效果最佳。在其他情况下，由于提升值、词频和长度归一化对分数的贡献方式，混合的词统计信息可能无法产生有意义的得分。

> fuzziness 参数不适用于 cross_fields 查询。

### 分词器的作用

`cross_fields` 查询仅能在具有相同分词器的字段上作为基于词条的查询工作。具有相同分词器的字段会被分组，这些分组会通过布尔查询组合在一起。

例如，考虑一个索引，其中 `first_name` 和 `last_name` 字段使用默认的 `standard` 分词器进行分析，而它们的 `.edge` 子字段使用边缘 n 元分词器进行分析：

```
PUT customers
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "first_name": {
        "type": "text",
        "fields": {
          "edge": {
            "type": "text",
            "analyzer": "my_analyzer"
          }
        }
      },
      "last_name": {
        "type": "text",
        "fields": {
          "edge": {
            "type": "text",
            "analyzer": "my_analyzer"
          }
        }
      }
    }
  }
}

```

你在 `customers` 索引中索引了一个文档：

```
PUT /customers/_doc/1
{
  "first_name": "John",
  "last_name": "Doe"
}
```

你可以使用 `cross_fields` 查询来跨字段搜索 `John Doe` :

```
GET /customers/_search
{
  "query": {
    "multi_match" : {
      "query": "John",
      "type": "cross_fields",
      "fields": [
        "first_name", "first_name.edge",
        "last_name",  "last_name.edge"
      ]
    }
  }
}

```

要查看查询的执行情况，你可以运行验证 API：

```
GET /customers/_validate/query?explain
{
  "query": {
    "multi_match" : {
      "query": "John",
      "type": "cross_fields",
      "fields": [
        "first_name", "first_name.edge",
        "last_name",  "last_name.edge"
      ]
    }
  }
}

```

返回显示 `last_name` 和 `first_name` 字段被组合在一起并视为一个字段。类似地， `last_name.edge` 和 `first_name.edge` 字段被组合在一起并视为一个字段：

```
{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "valid": true,
  "explanations": [
    {
      "index": "customers",
      "valid": true,
      "explanation": "(blended(terms:[last_name:john, first_name:john]) | (blended(terms:[last_name.edge:Jo, first_name.edge:Jo]) blended(terms:[last_name.edge:Joh, first_name.edge:Joh]) blended(terms:[last_name.edge:John, first_name.edge:John])))"
    }
  ]
}
```

使用 `operator` 或 `minimum_should_match` 参数与多个字段组（如前文所述）可能会导致前一节中描述的问题。为避免这种情况，你可以将之前的查询重写为两个 `cross_fields` 子查询，通过布尔查询将它们组合起来，并将 `minimum_should_match` 应用于其中一个子查询：

```
GET /customers/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "multi_match": {
            "query": "John Doe",
            "type": "cross_fields",
            "fields": [
              "first_name",
              "last_name"
            ],
            "minimum_should_match": "1"
          }
        },
        {
          "multi_match": {
            "query": "John Doe",
            "type": "cross_fields",
            "fields": [
              "first_name.edge",
              "last_name.edge"
            ]
          }
        }
      ]
    }
  }
}

```

为所有字段创建一个分组，请在查询中指定一个分词器：

```
GET customers/_search
{
  "query": {
   "multi_match" : {
      "query": "John Doe",
      "type": "cross_fields",
      "analyzer": "standard",
      "fields": [ "first_name", "last_name", "*.edge" ]
    }
  }
}

```

在先前查询上运行 Validate API 显示了查询是如何执行的：

```
{
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "valid": true,
  "explanations": [
    {
      "index": "customers",
      "valid": true,
      "explanation": "blended(terms:[last_name.edge:john, last_name:john, first_name:john, first_name.edge:john]) blended(terms:[last_name.edge:doe, last_name:doe, first_name:doe, first_name.edge:doe])"
    }
  ]
}
```

## 短语

`phrase` 查询的行为与 `best_fields` 查询类似，但使用 `match_phrase` 查询而不是 `match` 查询。

以下是在 `best_fields` 部分描述的索引中的 `phrase` 查询示例：

```
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "northern lights",
      "type": "phrase",
      "fields": [ "title", "description" ]
    }
  }
}
```

前面的查询被执行为以下 `dis_max` 查询，每个字段都有一个 `match_phrase` 查询：

```
GET articles/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_phrase": { "title": "northern lights" }},
        { "match_phrase": { "description": "northern lights" }}
      ]
    }
  }
}
```

因为默认情况下， `phrase` 查询仅当词项按相同顺序出现时才匹配文本，结果中仅返回文档 1

```
{
  "took": 3,
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
    "max_score": 0.84407747,
    "hits": [
      {
        "_index": "articles",
        "_id": "1",
        "_score": 0.84407747,
        "_source": {
          "title": "Aurora borealis",
          "description": "Northern lights, or aurora borealis, explained"
        }
      }
    ]
  }
}
```

您可以使用 `slop` 参数来允许查询短语中的词之间包含其他词。例如，以下查询在 `flourescent` 和 `therapy` 之间最多包含两个词时接受文本匹配：

```
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "fluorescent therapy",
      "type": "phrase",
      "fields": [ "title", "description" ],
      "slop": 2
    }
  }
}

```

返回包含文档 2

```
{
  "took": 3,
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
    "max_score": 0.7003825,
    "hits": [
      {
        "_index": "articles",
        "_id": "2",
        "_score": 0.7003825,
        "_source": {
          "title": "Sun deprivation in the Northern countries",
          "description": "Using fluorescent lights for therapy"
        }
      }
    ]
  }
}
```

对于 `slop` 值小于 2 的情况，不会返回任何文档。

> fuzziness 参数不适用于 phrase 查询。

## 短语前缀

`phrase_prefix` 查询的行为与 `phrase` 查询类似，但使用 `match_phrase_prefix` 查询而不是 `match_phrase` 查询。

以下是在 `best_fields` 部分描述的索引中的 `phrase_prefix` 查询示例：

```
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "northern light",
      "type": "phrase_prefix",
      "fields": [ "title", "description" ]
    }
  }
}
```

前面的查询被执行为以下 `dis_max` 查询，每个字段都有一个 `match_phrase_prefix` 查询：

```
GET articles/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_phrase_prefix": { "title": "northern light" }},
        { "match_phrase_prefix": { "description": "northern light" }}
      ]
    }
  }
}
```

您可以使用 `slop` 参数来允许在查询短语中的单词之间插入其他单词。

> fuzziness 参数不适用于 phrase_prefix 查询。

## 布尔前缀

`bool_prefix` 查询与 `most_fields` 查询类似地评分文档，但使用 `match_bool_prefix` 查询而不是 `match` 查询。

以下是在 `best_fields` 部分描述的索引中的 `bool_prefix` 查询示例：

```
GET articles/_search
{
  "query": {
    "multi_match" : {
      "query": "li northern",
      "type": "bool_prefix",
      "fields": [ "title", "description" ]
    }
  }
}

```

前面的查询被执行为以下 `dis_max` 查询，每个字段都有一个 `match_bool_prefix` 查询：

```
GET articles/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_bool_prefix": { "title": "li northern" }},
        { "match_bool_prefix": { "description": "li northern" }}
      ]
    }
  }
}
```

> 用于构建 terms 查询的 fuzziness 、 prefix_length 、 max_expansions 、 fuzzy_rewrite 和 fuzzy_transpositions 参数是受支持的，但它们不会对由最终 term 构建的前缀查询产生影响。

## 参数说明

该查询接受以下参数。除 `query` 外的所有参数都是可选的。

| 参数                                  | 数据类型                                 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `query`                               | String                                   | 用于搜索的查询字符串。必填。                                                                                                                                                                                                                                                                                                                                                                                           |
| `auto_generate_synonyms_phrase_query` | Boolean                                  | 指定是否自动为多词同义词创建匹配短语查询。例如，如果你将 ba,batting average 指定为同义词并搜索 ba ，Easysearch 会搜索 ba OR "batting average" （如果此选项为 true ）或 ba OR (batting AND average) （如果此选项为 false ）。默认值为 true 。                                                                                                                                                                           |
| `analyzer`                            | String                                   | 用于对查询字符串文本进行分词的分词器。默认值为为 default_field 指定的索引时分词器。如果未为 default_field 指定分词器，则 analyzer 为索引的默认分词器。有关 index.query.default_field 的更多信息，请参阅动态索引级索引设置。                                                                                                                                                                                            |
| `boost`                               | Float                                    | 通过给定的乘数提升子句的权重。适用于在复合查询中对子句进行加权。[0, 1) 范围内的值会降低相关性，而大于 1 的值会增加相关性。默认值为 1 。                                                                                                                                                                                                                                                                                |
| `fields`                              | 字符串数组                               | 要搜索的字段列表。如果你不提供 fields 参数， multi_match 查询会在 index.query.default_field 设置中指定的字段进行搜索，而 index.query.default_field 的默认值为 \* 。                                                                                                                                                                                                                                                    |
| `fuzziness`                           | String                                   | 在确定一个词是否匹配一个值时，将一个词改为另一个词所需的字符编辑次数（插入、删除、替换）。例如， wined 和 wind 之间的距离是 1。有效值为非负整数或 AUTO 。默认值 AUTO 根据每个词的长度选择值，对于大多数用例是一个不错的选择。不支持 phrase 、 phrase_prefix 和 cross_fields 查询。                                                                                                                                     |
| `fuzzy_rewrite`                       | String                                   | 确定 Easysearch 如何重写查询。有效值为 `constant_score`、`scoring_boolean`、`constant_score_boolean`、`top_terms_N`、`top_terms_boost_N` 和 `top_terms_blended_freqs_N`。如果 `fuzziness` 参数不是 0，查询默认使用 `top_terms_blended_freqs_${max_expansions}` 重写方法。默认值为 `constant_score`。                                                                                                             |
| `fuzzy_transpositions`                | Boolean                                  | 将 fuzzy_transpositions 设置为 true （默认）会在 fuzziness 选项的插入、删除和替换操作中添加相邻字符的交换。例如，如果 fuzzy_transpositions 为真（交换“n”和“i”），则 wind 和 wnid 之间的距离为 1；如果为假（删除“n”，插入“n”），则距离为 2。如果 fuzzy_transpositions 为假， rewind 和 wnid 与 wind 的距离相同（2），尽管从更以人为中心的观点来看， wnid 是一个明显的拼写错误。对于大多数用例，默认值是一个不错的选择。 |
| `lenient`                             | Boolean                                  | 将 lenient 设置为 true 会忽略查询与文档字段之间的数据类型不匹配。例如， "8.2" 的查询字符串可以匹配类型为 float 的字段。默认值为 false 。                                                                                                                                                                                                                                                                               |
| `max_expansions`                      | 正整数                                   | 查询可以扩展到的最大词数。模糊查询会扩展到与指定距离（ fuzziness ）内的匹配词。然后 Easysearch 尝试匹配这些词。默认值为 50 。                                                                                                                                                                                                                                                                                          |
| `minimum_should_match`                | 正整数或负整数、正百分比或负百分比、组合 | 如果查询字符串包含多个搜索词并且你使用 or 运算符，文档被考虑为匹配所需的匹配词数。例如，如果 minimum_should_match 为 2， wind often rising 不匹配 The Wind Rises. 如果 minimum_should_match 为 1 ，则匹配。详情请参阅 Minimum should match。                                                                                                                                                                           |
| `operator`                            | String                                   | 如果查询字符串包含多个搜索词，文档是否需要所有词都匹配（ AND ）或只需一个词匹配（ OR ）才被视为匹配。有效值为：- OR : 字符串 to be 被解释为 to OR be ;- AND : 字符串 to be 被解释为 to AND be.默认为 OR 。                                                                                                                                                                                                             |
| `prefix_length`                       | 非负整数                                 | 不考虑模糊性的前导字符数量。默认为 0 。                                                                                                                                                                                                                                                                                                                                                                                |
| `slop`                                | 0 （默认）或一个正整数                   | 控制查询中单词可以错乱的程度，仍然被视为匹配的程度。来自 Lucene 文档的说明：“查询短语中允许的其他单词数量。例如，要交换两个单词的顺序需要两次移动（第一次移动将单词叠放在一起），因此为了允许短语的重排序，slop 值必须至少为 2。值为零要求完全匹配。” 支持 phrase 和 phrase_prefix 查询类型。                                                                                                                          |
| `tie_breaker`                         | Float                                    | 一个介于 0 和 1.0 之间的因子，用于给匹配多个查询子句的文档赋予更高的权重。更多信息，请参阅`tie_breaker`参数。                                                                                                                                                                                                                                                                                                          |
| `type`                                | String                                   | 多匹配查询类型。有效值为`best_fields`、`most_fields`、`cross_fields`、`phrase`、`phrase_prefix`、`bool_prefix`。默认值为`best_fields`。                                                                                                                                                                                                                                                                                |
| `zero_terms_query`                    | String                                   | 在某些情况下，分词器会从查询字符串中删除所有词项。例如， stop 分词器会从字符串 an but this 中删除所有词项。在这种情况下， zero_terms_query 指定是否匹配不匹配任何文档（ none ）或所有文档（ all ）。有效值为 none 和 all 。默认为 none 。                                                                                                                                                                              |

> fuzziness 参数不适用于 phrase 、 phrase_prefix 和 cross_fields 查询。

> slop 参数仅适用于 phrase 和 phrase_prefix 查询。

### tie_breaker 参数

每个词级混合查询将文档分数计算为组中任何字段返回的最佳分数。所有混合查询的分数相加产生最终分数。您可以通过使用 `tie_breaker` 参数更改分数的计算方式。 `tie_breaker` 参数接受以下值：

- 0.0（ `best_fields` 、 `cross_fields` 、 `phrase` 和 `phrase_prefix` 查询的默认值）：取组中任何字段返回的最佳单个分数。
- 1.0（ `most_fields` 和 `bool_prefix` 查询的默认值）：为组内所有字段添加分数。
- 一个在(0, 1)范围内的浮点值：取最佳匹配字段的最佳分数，并为所有其他匹配字段添加( `tie_breaker` \* `_score` )。

