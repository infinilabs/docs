---
title: "Span Containing 查询"
date: 0001-01-01
summary: "Span Containing 查询 #  span_containing 查询会在较大的文本模式（如短语或一组单词）的边界内查找包含较小文本模式的匹配项。可以将其视为仅在特定更大的上下文中出现时才查找单词或短语。
例如，您可以使用 span_containing 查询来执行以下搜索：
 查找单词&quot;quick&quot;，但仅当它出现在同时提到狐狸和行为的句子中时。 确保某些词项出现在其他词项的上下文中——而不仅仅是在文档的任何地方。 搜索在更长的有意义的短语中出现的特定单词。  相关指南（先读这些） #    Span 查询  邻近匹配  查询 DSL 基础  参考样例 #  以下查询搜索在包含“silk”和“dress”词语的较大词组（不一定按该顺序）中，与“red”一词出现且彼此之间不超过 5 个词的情况：
GET /clothing/_search { &#34;query&#34;: { &#34;span_containing&#34;: { &#34;little&#34;: { &#34;span_term&#34;: { &#34;description&#34;: &#34;red&#34; } }, &#34;big&#34;: { &#34;span_near&#34;: { &#34;clauses&#34;: [ { &#34;span_term&#34;: { &#34;description&#34;: &#34;silk&#34; } }, { &#34;span_term&#34;: { &#34;description&#34;: &#34;dress&#34; } } ], &#34;slop&#34;: 5, &#34;in_order&#34;: false } } } } } 该查询匹配文档 1 的原因是："
---


# Span Containing 查询

`span_containing` 查询会在较大的文本模式（如短语或一组单词）的边界内查找包含较小文本模式的匹配项。可以将其视为仅在特定更大的上下文中出现时才查找单词或短语。

例如，您可以使用 `span_containing` 查询来执行以下搜索：

- 查找单词"quick"，但仅当它出现在同时提到狐狸和行为的句子中时。
- 确保某些词项出现在其他词项的上下文中——而不仅仅是在文档的任何地方。
- 搜索在更长的有意义的短语中出现的特定单词。

## 相关指南（先读这些）

- [Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}})
- [邻近匹配]({{< relref "/docs/features/fulltext-search/proximity-matching.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

以下查询搜索在包含“silk”和“dress”词语的较大词组（不一定按该顺序）中，与“red”一词出现且彼此之间不超过 5 个词的情况：

```
GET /clothing/_search
{
  "query": {
    "span_containing": {
      "little": {
        "span_term": {
          "description": "red"
        }
      },
      "big": {
        "span_near": {
          "clauses": [
            {
              "span_term": {
                "description": "silk"
              }
            },
            {
              "span_term": {
                "description": "dress"
              }
            }
          ],
          "slop": 5,
          "in_order": false
        }
      }
    }
  }
}
```

该查询匹配文档 1 的原因是：

- 它找到一个词组，其中“silk”和“dress”出现且彼此之间不超过 5 个词（“…dress in red silk…”）。词语“silk”和“dress”彼此之间相隔 2 个词（它们之间有 2 个词）。
- 在这个更大的跨度内，它找到了“red”这个词。

```
{
  "took": 4,
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
    "max_score": 1.1577396,
    "hits": [
      {
        "_index": "clothing",
        "_id": "2",
        "_score": 1.1577396,
        "_source": {
          "description": "Beautiful long dress in red silk, perfect for formal events."
        }
      }
    ]
  }
}
```

`little` 和 `big` 参数都可以包含任何类型的跨度查询，当需要时，允许进行复杂的嵌套跨度查询。

## 参数说明

下表列出了 `span_containing` 查询支持的所有顶层参数。所有参数都是必需的。

| 参数     | 数据类型 | 描述                                                                              |
| -------- | -------- | --------------------------------------------------------------------------------- |
| `little` | Object   | 必须包含在 `big` 跨度内的跨度查询。这定义了你在较大上下文中搜索的跨度。           |
| `big`    | Object   | 包含 span 查询，用于定义 `little` span 必须出现的边界。这为您的搜索建立了上下文。 |

