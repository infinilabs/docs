---
title: "Span Not 查询"
date: 0001-01-01
summary: "Span Not 查询 #  span_not 查询会排除与另一个 span 查询重叠的跨度。您还可以指定在排除的跨度之前或之后不允许匹配的距离范围。
例如，您可以使用 span_not 查询来：
 查找除在特定短语中出现时的词项外。 除非它们靠近特定词项，否则匹配跨度。 排除在特定距离内出现的其他模式匹配。  相关指南（先读这些） #    Span 查询  查询 DSL 基础  参考样例 #  以下查询搜索单词“dress”，但当它出现在短语“dress shirt”中时不搜索：
GET /clothing/_search { &#34;query&#34;: { &#34;span_not&#34;: { &#34;include&#34;: { &#34;span_term&#34;: { &#34;description&#34;: &#34;dress&#34; } }, &#34;exclude&#34;: { &#34;span_near&#34;: { &#34;clauses&#34;: [ { &#34;span_term&#34;: { &#34;description&#34;: &#34;dress&#34; } }, { &#34;span_term&#34;: { &#34;description&#34;: &#34;shirt&#34; } } ], &#34;slop&#34;: 0, &#34;in_order&#34;: true } } } } } 该查询匹配文档 2，因为它包含单词“dress”（“Beautiful long dress…”）。文档 1 未匹配，因为它包含短语“dress shirt”，该短语被排除。文档 3 和 4 未匹配，因为它们包含单词“dress”的变体（“dressed”和“dresses”），并且查询是在原始字段中进行的。"
---


# Span Not 查询

`span_not` 查询会排除与另一个 `span` 查询重叠的跨度。您还可以指定在排除的跨度之前或之后不允许匹配的距离范围。

例如，您可以使用 `span_not` 查询来：

- 查找除在特定短语中出现时的词项外。
- 除非它们靠近特定词项，否则匹配跨度。
- 排除在特定距离内出现的其他模式匹配。

## 相关指南（先读这些）

- [Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

以下查询搜索单词“dress”，但当它出现在短语“dress shirt”中时不搜索：

```
GET /clothing/_search
{
  "query": {
    "span_not": {
      "include": {
        "span_term": {
          "description": "dress"
        }
      },
      "exclude": {
        "span_near": {
          "clauses": [
            {
              "span_term": {
                "description": "dress"
              }
            },
            {
              "span_term": {
                "description": "shirt"
              }
            }
          ],
          "slop": 0,
          "in_order": true
        }
      }
    }
  }
}
```

该查询匹配文档 2，因为它包含单词“dress”（“Beautiful long dress…”）。文档 1 未匹配，因为它包含短语“dress shirt”，该短语被排除。文档 3 和 4 未匹配，因为它们包含单词“dress”的变体（“dressed”和“dresses”），并且查询是在原始字段中进行的。

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
    "max_score": 0.94519633,
    "hits": [
      {
        "_index": "clothing",
        "_type": "_doc",
        "_id": "2",
        "_score": 0.94519633,
        "_source": {
          "description": "Beautiful long dress in red silk, perfect for formal events."
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 `span_not` 查询支持的所有顶层参数。

| 参数      | 数据类型 | 描述                                                                                |
| --------- | -------- | ----------------------------------------------------------------------------------- |
| `include` | Object   | 你想查找匹配的 span 查询。必填。                                                    |
| `exclude` | Object   | 应该排除匹配的 span 查询。必填。                                                    |
| `pre`     | Integer  | 指定 exclude span 不能出现在 include span 之前的指定词元位置数内。可选。默认为 0 。 |
| `post`    | Integer  | 指定 exclude span 不能出现在 include span 之后的指定词元位置数内。可选。默认为 0 。 |
| `dist`    | Integer  | 相当于将 pre 和 post 设置为相同的值。可选。                                         |

