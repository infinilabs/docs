---
title: "Span Or 查询"
date: 0001-01-01
summary: "Span Or 查询 #  span_or 查询组合多个 span 查询，并匹配它们 span 的并集。如果其中至少一个包含的 span 查询匹配，则发生匹配。
例如，您可以使用 span_or 查询来：
 查找匹配多个模式中的任意一个的 span。 将不同的 span 模式作为备选项组合。 在一个查询中匹配多个 span 变体。  相关指南（先读这些） #    Span 查询  查询 DSL 基础  参考样例 #  以下查询搜索“formal collar”或“button collar”在彼此 2 个词距离内出现：
GET /clothing/_search { &#34;query&#34;: { &#34;span_or&#34;: { &#34;clauses&#34;: [ { &#34;span_near&#34;: { &#34;clauses&#34;: [ { &#34;span_term&#34;: { &#34;description&#34;: &#34;formal&#34; } }, { &#34;span_term&#34;: { &#34;description&#34;: &#34;collar&#34; } } ], &#34;slop&#34;: 0, &#34;in_order&#34;: true } }, { &#34;span_near&#34;: { &#34;clauses&#34;: [ { &#34;span_term&#34;: { &#34;description&#34;: &#34;button&#34; } }, { &#34;span_term&#34;: { &#34;description&#34;: &#34;collar&#34; } } ], &#34;slop&#34;: 2, &#34;in_order&#34;: true } } ] } } } 该查询在指定的 slop 距离内匹配文档 1（“…formal collar…”）和文档 3（“…button-down collar…”）。"
---


# Span Or 查询

`span_or` 查询组合多个 span 查询，并匹配它们 span 的并集。如果其中至少一个包含的 span 查询匹配，则发生匹配。

例如，您可以使用 `span_or` 查询来：

- 查找匹配多个模式中的任意一个的 span。
- 将不同的 span 模式作为备选项组合。
- 在一个查询中匹配多个 span 变体。

## 相关指南（先读这些）

- [Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

以下查询搜索“formal collar”或“button collar”在彼此 2 个词距离内出现：

```
GET /clothing/_search
{
  "query": {
    "span_or": {
      "clauses": [
        {
          "span_near": {
            "clauses": [
              {
                "span_term": {
                  "description": "formal"
                }
              },
              {
                "span_term": {
                  "description": "collar"
                }
              }
            ],
            "slop": 0,
            "in_order": true
          }
        },
        {
          "span_near": {
            "clauses": [
              {
                "span_term": {
                  "description": "button"
                }
              },
              {
                "span_term": {
                  "description": "collar"
                }
              }
            ],
            "slop": 2,
            "in_order": true
          }
        }
      ]
    }
  }
}
```

该查询在指定的 slop 距离内匹配文档 1（“…formal collar…”）和文档 3（“…button-down collar…”）。

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
      "value": 2,
      "relation": "eq"
    },
    "max_score": 2.170027,
    "hits": [
      {
        "_index": "clothing",
        "_id": "1",
        "_score": 2.170027,
        "_source": {
          "description": "Long-sleeved dress shirt with a formal collar and button cuffs. "
        }
      },
      {
        "_index": "clothing",
        "_id": "3",
        "_score": 1.2509141,
        "_source": {
          "description": "Short-sleeved shirt with a button-down collar, can be dressed up or down."
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 `span_or` 查询支持的所有顶层参数。

| 参数      | 数据类型 | 描述                                                                                                         |
| --------- | -------- | ------------------------------------------------------------------------------------------------------------ |
| `clauses` | Array    | 用于匹配的 span 查询数组。如果这些 span 查询中的任意一个匹配，则查询匹配。必须至少包含一个 span 查询。必填。 |

