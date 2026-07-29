---
title: "Constant Score 查询"
date: 0001-01-01
summary: "Constant Score 查询 #  如果您需要返回包含某个词的文档，而不管该词出现多少次，您可以使用 constant_score 查询。constant_score 查询包装一个过滤器查询，并将结果中的所有文档的关联分数设置为 boost 参数的值。因此，所有返回的文档具有相同的关联分数，并且不考虑词频/逆文档频率（TF/IDF）。过滤器查询不会计算关联分数。此外，Easysearch 会缓存常用的过滤器查询以提高性能。
相关指南（先读这些） #    查询 DSL 基础  结构化搜索  参考样例 #  使用以下查询返回 shakespeare 索引中包含单词“Hamlet”的文档：
GET shakespeare/_search { &#34;query&#34;: { &#34;constant_score&#34;: { &#34;filter&#34;: { &#34;match&#34;: { &#34;text_entry&#34;: &#34;Hamlet&#34; } }, &#34;boost&#34;: 1.2 } } } 结果中的所有文档都被分配了 1.2 的相关性分数：
{ &#34;took&#34;: 8, &#34;timed_out&#34;: false, &#34;_shards&#34;: { &#34;total&#34;: 1, &#34;successful&#34;: 1, &#34;skipped&#34;: 0, &#34;failed&#34;: 0 }, &#34;hits&#34;: { &#34;total&#34;: { &#34;value&#34;: 96, &#34;relation&#34;: &#34;eq&#34; }, &#34;max_score&#34;: 1."
---


# Constant Score 查询

如果您需要返回包含某个词的文档，而不管该词出现多少次，您可以使用 `constant_score` 查询。`constant_score` 查询包装一个过滤器查询，并将结果中的所有文档的关联分数设置为 `boost` 参数的值。因此，所有返回的文档具有相同的关联分数，并且不考虑词频/逆文档频率（TF/IDF）。过滤器查询不会计算关联分数。此外，Easysearch 会缓存常用的过滤器查询以提高性能。

## 相关指南（先读这些）

- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

## 参考样例

使用以下查询返回 `shakespeare` 索引中包含单词“Hamlet”的文档：

```
GET shakespeare/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "match": {
          "text_entry": "Hamlet"
        }
      },
      "boost": 1.2
    }
  }
}
```

结果中的所有文档都被分配了 1.2 的相关性分数：

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
      "value": 96,
      "relation": "eq"
    },
    "max_score": 1.2,
    "hits": [
      {
        "_index": "shakespeare",
        "_id": "32535",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32536,
          "play_name": "Hamlet",
          "speech_number": 48,
          "line_number": "1.1.97",
          "speaker": "HORATIO",
          "text_entry": "Dared to the combat; in which our valiant Hamlet--"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32546",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32547,
          "play_name": "Hamlet",
          "speech_number": 48,
          "line_number": "1.1.108",
          "speaker": "HORATIO",
          "text_entry": "His fell to Hamlet. Now, sir, young Fortinbras,"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32625",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32626,
          "play_name": "Hamlet",
          "speech_number": 59,
          "line_number": "1.1.184",
          "speaker": "HORATIO",
          "text_entry": "Unto young Hamlet; for, upon my life,"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32633",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32634,
          "play_name": "Hamlet",
          "speech_number": 60,
          "line_number": "",
          "speaker": "MARCELLUS",
          "text_entry": "Enter KING CLAUDIUS, QUEEN GERTRUDE, HAMLET,  POLONIUS, LAERTES, VOLTIMAND, CORNELIUS, Lords, and Attendants"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32634",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32635,
          "play_name": "Hamlet",
          "speech_number": 1,
          "line_number": "1.2.1",
          "speaker": "KING CLAUDIUS",
          "text_entry": "Though yet of Hamlet our dear brothers death"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32699",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32700,
          "play_name": "Hamlet",
          "speech_number": 8,
          "line_number": "1.2.65",
          "speaker": "KING CLAUDIUS",
          "text_entry": "But now, my cousin Hamlet, and my son,--"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32703",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32704,
          "play_name": "Hamlet",
          "speech_number": 12,
          "line_number": "1.2.69",
          "speaker": "QUEEN GERTRUDE",
          "text_entry": "Good Hamlet, cast thy nighted colour off,"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32723",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32724,
          "play_name": "Hamlet",
          "speech_number": 16,
          "line_number": "1.2.89",
          "speaker": "KING CLAUDIUS",
          "text_entry": "Tis sweet and commendable in your nature, Hamlet,"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32754",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32755,
          "play_name": "Hamlet",
          "speech_number": 17,
          "line_number": "1.2.120",
          "speaker": "QUEEN GERTRUDE",
          "text_entry": "Let not thy mother lose her prayers, Hamlet:"
        }
      },
      {
        "_index": "shakespeare",
        "_id": "32759",
        "_score": 1.2,
        "_source": {
          "type": "line",
          "line_id": 32760,
          "play_name": "Hamlet",
          "speech_number": 19,
          "line_number": "1.2.125",
          "speaker": "KING CLAUDIUS",
          "text_entry": "This gentle and unforced accord of Hamlet"
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 `constant_score` 查询支持的所有顶层参数。

| 参数     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| `filter` | 文档必须匹配的过滤器查询才能在结果中返回。必填。             |
| `boost`  | 分配给所有返回文档的相关性分数的浮点值。可选。默认值为 1.0。 |

