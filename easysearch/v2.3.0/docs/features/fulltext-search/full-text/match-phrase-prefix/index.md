---
title: "Match Phrase Prefix 查询"
date: 0001-01-01
summary: "Match Phrase Prefix 查询 #  使用 match_phrase_prefix 查询来指定要匹配的短语。包含您指定短语的文档将被返回。短语中的最后一个部分词被解释为前缀，因此任何包含以该短语和最后一个词的前缀开头的短语的文档都将被返回。
与 match_phrase 类似，但会从查询字符串中的最后一个词创建一个前缀查询。
相关指南（先读这些） #    部分匹配  邻近匹配  对于 match_phrase_prefix 和 match_bool_prefix 查询之间的差异，请参阅 match_bool_prefix 和 match_phrase_prefix 查询。
以下示例展示了一个基本的 match_phrase_prefix 查询：
GET _search { &#34;query&#34;: { &#34;match_phrase_prefix&#34;: { &#34;title&#34;: &#34;the wind&#34; } } } 要传递附加参数，您可以使用扩展语法：
GET _search { &#34;query&#34;: { &#34;match_phrase_prefix&#34;: { &#34;title&#34;: { &#34;query&#34;: &#34;the wind&#34;, &#34;analyzer&#34;: &#34;stop&#34; } } } } 参考用例 #  例如，创建一个包含以下文档的索引：
PUT testindex/_doc/1 { &#34;title&#34;: &#34;The wind rises&#34; } PUT testindex/_doc/2 { &#34;title&#34;: &#34;Gone with the wind&#34; } 以下 match_phrase_prefix 查询会搜索完整单词 wind ，后跟一个以 ri 开头的单词："
---


# Match Phrase Prefix 查询

使用 `match_phrase_prefix` 查询来指定要匹配的短语。包含您指定短语的文档将被返回。短语中的最后一个部分词被解释为前缀，因此任何包含以该短语和最后一个词的前缀开头的短语的文档都将被返回。

与 `match_phrase` 类似，但会从查询字符串中的最后一个词创建一个前缀查询。

## 相关指南（先读这些）

- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [邻近匹配]({{< relref "/docs/features/fulltext-search/proximity-matching.md" >}})

对于 `match_phrase_prefix` 和 `match_bool_prefix` 查询之间的差异，请参阅 `match_bool_prefix` 和 `match_phrase_prefix` 查询。

以下示例展示了一个基本的 `match_phrase_prefix` 查询：

```
GET _search
{
  "query": {
    "match_phrase_prefix": {
      "title": "the wind"
    }
  }
}
```

要传递附加参数，您可以使用扩展语法：

```
GET _search
{
  "query": {
    "match_phrase_prefix": {
      "title": {
        "query": "the wind",
        "analyzer": "stop"
      }
    }
  }
}

```

## 参考用例

例如，创建一个包含以下文档的索引：

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

以下 `match_phrase_prefix` 查询会搜索完整单词 `wind` ，后跟一个以 `ri` 开头的单词：

```
GET testindex/_search
{
  "query": {
    "match_phrase_prefix": {
      "title": "wind ri"
    }
  }
}
```

返回包含匹配的文档：

```
{
  "took": 6,
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
    "max_score": 0.92980814,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.92980814,
        "_source": {
          "title": "The wind rises"
        }
      }
    ]
  }
}
```

## 参数说明

该查询将字段名称（ `<field>`）作为顶级参数接受：

```
GET _search
{
  "query": {
    "match_phrase_prefix": {
      "<field>": {
        "query": "text to search for",
        ...
      }
    }
  }
}
```

`<field>` 接受以下参数。除 `query` 外，所有参数都是可选的。

| 参数             | 数据类型               | 描述                                                                                                                                                                                                                                                          |
| ---------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `query`          | String                 | 用于搜索的查询字符串。必填。                                                                                                                                                                                                                                  |
| `analyzer`       | String                 | 用于分词查询的分词器。                                                                                                                                                                                                                                        |
| `max_expansions` | Positive integer       | 最后一个词项（前缀）可以扩展匹配的最大词项数。值越大召回越多但性能越低。默认值为 `50`。 |
| `slop`           | 0（默认）或正整数 | 允许匹配词项之间插入的其他词项数量。例如，交换两个词项的顺序需要 slop 为 2。值为 0 要求完全匹配短语顺序。 |
| `zero_terms_query` | String              | 当分析器移除查询字符串中所有词项时（如停用词过滤），指定匹配行为。`none` 不匹配任何文档，`all` 匹配所有文档。默认 `none`。 |
| `boost`          | Float                  | 用于调整该查询的相关性得分权重。默认值为 `1.0`。 |
