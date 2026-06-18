---
title: "Span Near 查询"
date: 0001-01-01
summary: "Span Near 查询 #  span_near 查询匹配彼此靠近的跨度。您可以指定跨度之间的距离，并指定它们是否需要按特定顺序出现。
例如，您可以使用 span_near 查询来：
 查找彼此之间距离在特定范围内的词项。 匹配词语按特定顺序出现的短语。 查找文本中彼此靠近的相关概念。  相关指南（先读这些） #    Span 查询  邻近匹配  查询 DSL 基础  参考样例 #  以下查询搜索任何形式的“sleeve”和“long”相邻出现，顺序不限：
GET /clothing/_search { &#34;query&#34;: { &#34;span_near&#34;: { &#34;clauses&#34;: [ { &#34;span_term&#34;: { &#34;description.stemmed&#34;: &#34;sleev&#34; } }, { &#34;span_term&#34;: { &#34;description.stemmed&#34;: &#34;long&#34; } } ], &#34;slop&#34;: 1, &#34;in_order&#34;: false } } } 该查询匹配文档 1（“Long-sleeved…”）和文档 2（“…long fluttered sleeves…”）。在文档 1 中，词语是相邻的，而在文档 2 中，它们在指定的 slop 距离 1 内（它们之间有一个词）。"
---


# Span Near 查询

`span_near` 查询匹配彼此靠近的跨度。您可以指定跨度之间的距离，并指定它们是否需要按特定顺序出现。

例如，您可以使用 `span_near` 查询来：

- 查找彼此之间距离在特定范围内的词项。
- 匹配词语按特定顺序出现的短语。
- 查找文本中彼此靠近的相关概念。

## 相关指南（先读这些）

- [Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}})
- [邻近匹配]({{< relref "/docs/features/fulltext-search/proximity-matching.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

以下查询搜索任何形式的“sleeve”和“long”相邻出现，顺序不限：

```
GET /clothing/_search
{
  "query": {
    "span_near": {
      "clauses": [
        {
          "span_term": {
            "description.stemmed": "sleev"
          }
        },
        {
          "span_term": {
            "description.stemmed": "long"
          }
        }
      ],
      "slop": 1,
      "in_order": false
    }
  }
}
```

该查询匹配文档 1（“Long-sleeved…”）和文档 2（“…long fluttered sleeves…”）。在文档 1 中，词语是相邻的，而在文档 2 中，它们在指定的 `slop` 距离 1 内（它们之间有一个词）。

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
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.36496973,
    "hits": [
      {
        "_index": "clothing",
        "_id": "1",
        "_score": 0.36496973,
        "_source": {
          "description": "Long-sleeved dress shirt with a formal collar and button cuffs. "
        }
      },
      {
        "_index": "clothing",
        "_id": "4",
        "_score": 0.25312424,
        "_source": {
          "description": "A set of two midi silk shirt dresses with long fluttered sleeves in black. "
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 `span_near` 查询支持的所有顶层参数。

| 参数       | 数据类型 | 描述                                                                                           |
| ---------- | -------- | ---------------------------------------------------------------------------------------------- |
| `clauses`  | String   | 一组 span 查询，用于定义要匹配的词项或短语。所有指定的词项必须出现在定义的 slop 距离内。必填。 |
| `slop`     | Integer  | 跨度之间最大数量的未匹配位置。必填。                                                           |
| `in_order` | Boolean  | 跨度是否需要按照 `clauses` 数组中的顺序出现。可选。默认值为 false 。                           |

