---
title: "Span Within 查询"
date: 0001-01-01
summary: "Span Within 查询 #  span_within 查询匹配被另一个 span 查询所包围的跨度。它是 span_containing 的相反操作：span_containing 返回包含较小跨度的较大跨度，而 span_within 返回被较大跨度包围的较小跨度。
例如，您可以使用 span_within 查询来：
 查找出现在较长短语中的较短短语。 匹配在特定上下文中出现的词项。 识别被较大模式包围的小模式。  相关指南（先读这些） #    Span 查询  邻近匹配  查询 DSL 基础  参考样例 #  以下查询在包含“shirt”和“long”的跨度中搜索单词“dress”：
GET /clothing/_search { &#34;query&#34;: { &#34;span_within&#34;: { &#34;little&#34;: { &#34;span_term&#34;: { &#34;description&#34;: &#34;dress&#34; } }, &#34;big&#34;: { &#34;span_near&#34;: { &#34;clauses&#34;: [ { &#34;span_term&#34;: { &#34;description&#34;: &#34;shirt&#34; } }, { &#34;span_term&#34;: { &#34;description&#34;: &#34;long&#34; } } ], &#34;slop&#34;: 2, &#34;in_order&#34;: false } } } } } 该查询匹配文档 1 的原因是："
---


# Span Within 查询

`span_within` 查询匹配被另一个 span 查询所包围的跨度。它是 `span_containing` 的相反操作：`span_containing` 返回包含较小跨度的较大跨度，而 `span_within` 返回被较大跨度包围的较小跨度。

例如，您可以使用 `span_within` 查询来：

- 查找出现在较长短语中的较短短语。
- 匹配在特定上下文中出现的词项。
- 识别被较大模式包围的小模式。

## 相关指南（先读这些）

- [Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}})
- [邻近匹配]({{< relref "/docs/features/fulltext-search/proximity-matching.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

以下查询在包含“shirt”和“long”的跨度中搜索单词“dress”：

```
GET /clothing/_search
{
  "query": {
    "span_within": {
      "little": {
        "span_term": {
          "description": "dress"
        }
      },
      "big": {
        "span_near": {
          "clauses": [
            {
              "span_term": {
                "description": "shirt"
              }
            },
            {
              "span_term": {
                "description": "long"
              }
            }
          ],
          "slop": 2,
          "in_order": false
        }
      }
    }
  }
}
```

该查询匹配文档 1 的原因是：

- 单词“dress”出现在较大的跨度（“Long-sleeved dress shirt…”）中。
- 较大的跨度包含“shirt”和“long”，它们在彼此之间相隔 2 个词（它们之间有 2 个词）。

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
    "max_score": 1.4677674,
    "hits": [
      {
        "_index": "clothing",
        "_id": "1",
        "_score": 1.4677674,
        "_source": {
          "description": "Long-sleeved dress shirt with a formal collar and button cuffs. "
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 `span_within` 查询支持的所有顶层参数。所有参数都是必需的。

| 参数     | 数据类型 | 描述                                                                              |
| -------- | -------- | --------------------------------------------------------------------------------- |
| `little` | Object   | 必须包含在 `big` span 内的跨度查询。这定义了你在较大上下文中搜索的跨度。          |
| `big`    | Object   | 包含 span 查询，用于定义 `little` span 必须出现的边界。这为您的搜索建立了上下文。 |

