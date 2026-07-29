---
title: "Match Phrase 查询"
date: 0001-01-01
summary: "Match Phrase 查询 #  使用 match_phrase 查询来匹配包含指定顺序中确切的短语的文档。您可以通过提供 slop 参数来增加短语匹配的灵活性。
match_phrase 查询创建一个匹配词项序列的短语查询。
相关指南（先读这些） #    邻近匹配  全文搜索  以下示例展示了一个基本的 match_phrase 查询：
GET _search { &#34;query&#34;: { &#34;match_phrase&#34;: { &#34;title&#34;: &#34;the wind&#34; } } } 要传递额外的参数，您可以使用扩展语法：
GET _search { &#34;query&#34;: { &#34;match_phrase&#34;: { &#34;title&#34;: { &#34;query&#34;: &#34;the wind&#34;, &#34;analyzer&#34;: &#34;stop&#34; } } } } 参考用例 #  例如，创建一个包含以下文档的索引：
PUT testindex/_doc/1 { &#34;title&#34;: &#34;The wind rises&#34; } PUT testindex/_doc/2 { &#34;title&#34;: &#34;Gone with the wind&#34; } 以下 match_phrase 查询搜索短语 wind rises ，其中单词 wind 后面跟着单词 rises ："
---


# Match Phrase 查询

使用 `match_phrase` 查询来匹配包含指定顺序中确切的短语的文档。您可以通过提供 `slop` 参数来增加短语匹配的灵活性。

`match_phrase` 查询创建一个匹配词项序列的短语查询。

## 相关指南（先读这些）

- [邻近匹配]({{< relref "/docs/features/fulltext-search/proximity-matching.md" >}})
- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})

以下示例展示了一个基本的 `match_phrase` 查询：

```
GET _search
{
  "query": {
    "match_phrase": {
      "title": "the wind"
    }
  }
}
```

要传递额外的参数，您可以使用扩展语法：

```
GET _search
{
  "query": {
    "match_phrase": {
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

以下 `match_phrase` 查询搜索短语 `wind rises` ，其中单词 `wind` 后面跟着单词 `rises` ：

```
GET testindex/_search
{
  "query": {
    "match_phrase": {
      "title": "wind rises"
    }
  }
}
```

返回内容包含匹配的文档：

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

## 分词器

默认情况下，当你对 `text` 字段运行查询时，搜索文本会使用与该字段关联的索引分词器进行分析。你可以在 `analyzer` 参数中指定不同的搜索分词器。例如，以下查询使用了 `english` 分词器：

```
GET testindex/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "the winds",
        "analyzer": "english"
      }
    }
  }
}
```

`english` 分词器移除了停用词 `the` 并执行词干提取，生成了词元 `wind` 。两个文档都匹配这个词元，并在结果中返回：

```
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
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.19363807,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.19363807,
        "_source": {
          "title": "The wind rises"
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 0.17225474,
        "_source": {
          "title": "Gone with the wind"
        }
      }
    ]
  }
}
```

## Slop

如果你提供一个 `slop` 参数，查询会容忍搜索词的重新排序。`Slop` 指定了查询短语中允许在两个词之间插入的其他词的数量。例如，在以下查询中，搜索文本与文档文本相比进行了重新排序：

```
GET _search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "wind rises the",
        "slop": 3
      }
    }
  }
}
```

查询仍然返回匹配的文档：

```
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
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.44026947,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0.44026947,
        "_source": {
          "title": "The wind rises"
        }
      }
    ]
  }
}
```

## 参数说明

该查询将字段名称（ `<field>` ）作为顶级参数接受：

```
GET _search
{
  "query": {
    "match_phrase": {
      "<field>": {
        "query": "text to search for",
        ...
      }
    }
  }
}

```

`<field>` 接受以下参数。除 `query` 外，所有参数都是可选的。

| 参数               | 数据类型               | 描述                                                                                                                                                                                                                                                    |
| ------------------ | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `query`            | String                 | 用于搜索的查询字符串。必填。                                                                                                                                                                                                                            |
| `analyzer`         | String                 | 用于对查询字符串文本进行分词的分词器。默认值为为 default_field 指定的索引时分词器。如果未为 default_field 指定分词器，则 analyzer 为索引的默认分词器。有关 index.query.default_field 的更多信息，请参阅动态索引级索引设置。                             |
| `slop`             | 0 （默认）或一个正整数 | 控制查询中的词语可以错乱到何种程度仍被视为匹配。根据 Lucene 文档的说明：“查询短语中允许插入的其他词语数量。例如，要交换两个词语的顺序需要两次移动（第一次移动将词语叠放在一起），因此为了允许短语的重排序，slop 值必须至少为 2。值为零则要求完全匹配。” |
| `zero_terms_query` | String                 | 在某些情况下，分词器会从查询字符串中删除所有词项。例如， stop 分词器会从字符串 an but this 中删除所有词项。在这种情况下， zero_terms_query 指定是否匹配不匹配任何文档（ none ）或所有文档（ all ）。有效值为 none 和 all 。默认为 none 。               |
| `boost`            | Float                  | 用于调整该查询的相关性得分权重。默认值为 `1.0`。                                                                                                                                                                                                        |

