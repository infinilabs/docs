---
title: "Span First 查询"
date: 0001-01-01
summary: "Span First 查询 #  span_first 查询匹配从字段开头开始并在指定位置结束的跨度。当您想要查找出现在文档开头附近的词项或短语时，此查询很有用。
例如，您可以使用 span_first 查询来执行以下搜索：
 查找在字段的最初几个词中出现的特定词项的文档。 确保某些短语出现在文本的开头或附近。 仅在模式出现在指定距离内时匹配。  相关指南（先读这些） #    Span 查询  查询 DSL 基础  参考样例 #  以下查询搜索词干“dress”出现在描述的前 4 个位置：
GET /clothing/_search { &#34;query&#34;: { &#34;span_first&#34;: { &#34;match&#34;: { &#34;span_term&#34;: { &#34;description.stemmed&#34;: &#34;dress&#34; } }, &#34;end&#34;: 4 } } } 该查询匹配文档 1 和 2：
 文档 1 和 2 在第三位置包含单词 dress （“长袖连衣裙…”和“漂亮的连衣裙”）。索引单词的起始位置为 0，因此单词“dress”位于位置 2。 单词 dress 的位置必须小于 4 ，如 end 参数指定。  { &#34;took&#34;: 13, &#34;timed_out&#34;: false, &#34;_shards&#34;: { &#34;total&#34;: 1, &#34;successful&#34;: 1, &#34;skipped&#34;: 0, &#34;failed&#34;: 0 }, &#34;hits&#34;: { &#34;total&#34;: { &#34;value&#34;: 2, &#34;relation&#34;: &#34;eq&#34; }, &#34;max_score&#34;: 0."
---


# Span First 查询

`span_first` 查询匹配从字段开头开始并在指定位置结束的跨度。当您想要查找出现在文档开头附近的词项或短语时，此查询很有用。

例如，您可以使用 `span_first` 查询来执行以下搜索：

- 查找在字段的最初几个词中出现的特定词项的文档。
- 确保某些短语出现在文本的开头或附近。
- 仅在模式出现在指定距离内时匹配。

## 相关指南（先读这些）

- [Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

以下查询搜索词干“dress”出现在描述的前 4 个位置：

```
GET /clothing/_search
{
  "query": {
    "span_first": {
      "match": {
        "span_term": {
          "description.stemmed": "dress"
        }
      },
      "end": 4
    }
  }
}
```

该查询匹配文档 1 和 2：

- 文档 1 和 2 在第三位置包含单词 dress （“长袖连衣裙…”和“漂亮的连衣裙”）。索引单词的起始位置为 0，因此单词“dress”位于位置 2。
- 单词 dress 的位置必须小于 4 ，如 end 参数指定。

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
    "max_score": 0.110377684,
    "hits": [
      {
        "_index": "clothing",
        "_id": "1",
        "_score": 0.110377684,
        "_source": {
          "description": "Long-sleeved dress shirt with a formal collar and button cuffs. "
        }
      },
      {
        "_index": "clothing",
        "_id": "2",
        "_score": 0.110377684,
        "_source": {
          "description": "Beautiful long dress in red silk, perfect for formal events."
        }
      }
    ]
  }
}
```

`match` 参数可以包含任何类型的跨度查询，允许在字段的开始处匹配更复杂的模式。

## 参数说明

下表列出了 `span_first` 查询支持的所有顶层参数。所有参数都是必需的。

| 参数    | 数据类型 | 描述                                                                         |
| ------- | -------- | ---------------------------------------------------------------------------- |
| `match` | Object   | 用于匹配的跨度查询。这定义了你在字段开始处搜索的模式。                       |
| `end`   | Integer  | span 查询匹配允许的最大结束位置（不包含）。例如， end: 4 匹配位置 0–3 的词。 |

