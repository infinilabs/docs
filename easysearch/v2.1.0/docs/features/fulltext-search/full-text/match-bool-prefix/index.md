---
title: "Match Bool Prefix 查询"
date: 0001-01-01
summary: "Match Bool Prefix 查询 #  match_bool_prefix 查询分析提供的搜索字符串，并从字符串的词项中创建一个布尔查询。它将除最后一个词项外的每个词项作为完整单词进行匹配。最后一个词项用作前缀。match_bool_prefix 查询返回包含完整单词词项或以前缀词项开头的词项的文档，顺序不限。
相关指南（先读这些） #    部分匹配  全文搜索  以下示例展示了一个基本的 match_bool_prefix 查询：
GET _search { &#34;query&#34;: { &#34;match_bool_prefix&#34;: { &#34;title&#34;: &#34;the wind&#34; } } } 要传递额外参数，您可以使用扩展语法：
GET _search { &#34;query&#34;: { &#34;match_bool_prefix&#34;: { &#34;title&#34;: { &#34;query&#34;: &#34;the wind&#34;, &#34;analyzer&#34;: &#34;stop&#34; } } } } 参考样例 #  例如，考虑一个包含以下文档的索引：
PUT testindex/_doc/1 { &#34;title&#34;: &#34;The wind rises&#34; } PUT testindex/_doc/2 { &#34;title&#34;: &#34;Gone with the wind&#34; } 以下 match_bool_prefix 查询会搜索整个词 rises 以及以 wi 开头的词，顺序不限："
---


# Match Bool Prefix 查询

`match_bool_prefix` 查询分析提供的搜索字符串，并从字符串的词项中创建一个布尔查询。它将除最后一个词项外的每个词项作为完整单词进行匹配。最后一个词项用作前缀。`match_bool_prefix` 查询返回包含完整单词词项或以前缀词项开头的词项的文档，顺序不限。

## 相关指南（先读这些）

- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})

以下示例展示了一个基本的 `match_bool_prefix` 查询：

```
GET _search
{
  "query": {
    "match_bool_prefix": {
      "title": "the wind"
    }
  }
}

```

要传递额外参数，您可以使用扩展语法：

```
GET _search
{
  "query": {
    "match_bool_prefix": {
      "title": {
        "query": "the wind",
        "analyzer": "stop"
      }
    }
  }
}

```

## 参考样例

例如，考虑一个包含以下文档的索引：

```
PUT testindex/_doc/1
{
  "title": "The wind rises"
}

PUT testindex/_doc/2
{
  "title": "Gone with the wind"

}

```

以下 `match_bool_prefix` 查询会搜索整个词 `rises` 以及以 `wi` 开头的词，顺序不限：

```
GET testindex/_search
{
  "query": {
    "match_bool_prefix": {
      "title": "rises wi"
    }
  }
}
```

前面的查询等同于以下布尔查询：

```
GET testindex/_search
{
  "query": {
    "bool" : {
      "should": [
        { "term": { "title": "rises" }},
        { "prefix": { "title": "wi"}}
      ]
    }
  }
}
```

响应包含两个文档：

```
{
  "took": 15,
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
    "max_score": 1.73617,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1.73617,
        "_source": {
          "title": "The wind rises"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "Gone with the wind"
        }
      }
    ]
  }
}
```

## match_bool_prefix 和 match_phrase_prefix 查询

`match_bool_prefix` 查询匹配任何位置的词，而 `match_phrase_prefix` 查询匹配整个短语。为了说明区别，再次考虑上一节的 `match_bool_prefix` 查询：

```
GET testindex/_search
{
  "query": {
    "match_bool_prefix": {
      "title": "rises wi"
    }
  }
}
```

`The wind rises` 和 `Gone with the wind` 都匹配了搜索词，因此查询返回了这两份文档。

现在在同一索引上运行一个 `match_phrase_prefix` 查询：

```
GET testindex/_search
{
  "query": {
    "match_phrase_prefix": {
      "title": "rises wi"
    }
  }
}

```

响应返回没有文档，因为没有任何文档包含按指定顺序出现的 `rises wi` 短语。

## 分词器

默认情况下，当你在一个 `text` 字段上运行查询时，搜索文本会使用与该字段关联的索引分词器进行分析。你可以在 `analyzer` 参数中指定不同的搜索分词器：

```
GET testindex/_search
{
  "query": {
    "match_bool_prefix": {
      "title": {
        "query": "rise the wi",
        "analyzer": "stop"
      }
    }
  }
}

```

## 参数说明

该查询将字段名称（ `<field>` ）作为顶级参数接受：

```
GET _search
{
  "query": {
    "match_bool_prefix": {
      "<field>": {
        "query": "text to search for",
        ...
      }
    }
  }
}

```

`<field>` 接受以下参数。除 `query` 外，所有参数都是可选的。

| 参数                   | 数据类型                                             | 描述                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `query`                | String                                               | 用于搜索的文本、数字、布尔值或日期。必填。                                                                                                                                                                                                                                                                                                                                                                             |
| `analyzer`             | String                                               | 用于对查询字符串文本进行分词的分词器。默认值为为 default_field 指定的索引时分词器。如果未为 default_field 指定分词器，则 analyzer 为索引的默认分词器。有关 index.query.default_field 的更多信息，请参阅动态索引级索引设置。                                                                                                                                                                                            |
| `fuzziness`            | AUTO 、 0 或正整数                                   | 在确定一个词是否匹配一个值时，将一个词改为另一个词所需的字符编辑次数（插入、删除、替换）。例如， wined 和 wind 之间的距离是 1。默认值 AUTO 根据每个词的长度选择值，对于大多数用例是一个不错的选择。                                                                                                                                                                                                                    |
| `fuzzy_rewrite`        | String                                               | 确定 Easysearch 如何重写查询。有效值为 `constant_score`、`scoring_boolean`、`constant_score_boolean`、`top_terms_N`、`top_terms_boost_N` 和 `top_terms_blended_freqs_N`。如果 `fuzziness` 参数不是 0，查询默认使用 `top_terms_blended_freqs_${max_expansions}` 重写方法。默认值为 `constant_score`。                                                                                                             |
| `fuzzy_transpositions` | Boolean                                              | 将 `fuzzy_transpositions` 设置为 `true`（默认）会在 `fuzziness` 选项的插入、删除和替换操作中添加相邻字符的交换。例如，如果为 `true`（交换 "n" 和 "i"），则 `wind` 和 `wnid` 之间的距离为 1；如果为 `false`（删除 "n"，插入 "n"），则距离为 2。默认值为 `true`。 |
| `max_expansions`       | 正整数                                               | 查询可以扩展到的最大词数。模糊查询会扩展到与指定距离（ fuzziness ）内的匹配词。然后 Easysearch 尝试匹配这些词。默认值为 50 。                                                                                                                                                                                                                                                                                          |
| `minimum_should_match` | 正整数或负整数、正百分比或负百分比、或者这些类型组合 | 如果查询字符串包含多个搜索词并且你使用 or 运算符，文档被考虑为匹配所需的匹配词数。例如，如果 minimum_should_match 为 2， wind often rising 不匹配 The Wind Rises. 如果 minimum_should_match 为 1 ，则匹配。详情请参阅 Minimum should match。                                                                                                                                                                           |
| `operator`             | String                                               | 如果查询字符串包含多个搜索词，是否所有词都需要匹配（ and ）或只需要一个词匹配（ or ）才能认为文档匹配。有效值为 or 和 and 。默认值是 or 。                                                                                                                                                                                                                                                                             |
| `prefix_length`        | 非负整数                                             | 不考虑模糊性的前导字符数量。默认为 0 。                                                                                                                                                                                                                                                                                                                                                                                |
| `boost`                | Float                                                | 用于调整该查询的相关性得分权重。默认值为 `1.0`。                                                                                                                                                                                                                                                                                                                                                                       |

