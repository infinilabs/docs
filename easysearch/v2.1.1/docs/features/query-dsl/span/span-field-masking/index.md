---
title: "Span Field Masking 查询"
date: 0001-01-01
summary: "Span Field Masking 查询 #  field_masking_span 查询允许 span 查询通过&quot;掩饰&quot;查询的真实字段来匹配不同字段。这在处理多字段（相同内容使用不同分析器索引）或需要跨不同字段运行 span 查询（如 span_near 或 span_or，这通常是不允许的）时特别有用。
例如，您可以使用 field_masking_span 查询来：
 匹配原始字段及其词干版本中的词项。 在一个 span 操作中组合不同字段的 span 查询。 使用不同分析器索引的相同内容进行操作。   注意：在使用字段遮罩时，相关性分数是根据遮罩字段的特性（范数）计算的，而不是实际搜索的字段。这意味着如果遮罩字段与被搜索字段具有不同的属性（如长度或提升值），您可能会收到意外的评分结果。
 相关指南（先读这些） #    Span 查询  多字段搜索  查询 DSL 基础  参考样例 #  以下查询在词干化字段中搜索单词“long”，并查找“sleeve”一词的变体附近：
GET /clothing/_search { &#34;query&#34;: { &#34;span_near&#34;: { &#34;clauses&#34;: [ { &#34;span_term&#34;: { &#34;description&#34;: &#34;long&#34; } }, { &#34;field_masking_span&#34;: { &#34;query&#34;: { &#34;span_term&#34;: { &#34;description."
---


# Span Field Masking 查询

`field_masking_span` 查询允许 `span` 查询通过"掩饰"查询的真实字段来匹配不同字段。这在处理多字段（相同内容使用不同分析器索引）或需要跨不同字段运行 `span` 查询（如 `span_near` 或 `span_or`，这通常是不允许的）时特别有用。

例如，您可以使用 `field_masking_span` 查询来：

- 匹配原始字段及其词干版本中的词项。
- 在一个 `span` 操作中组合不同字段的 `span` 查询。
- 使用不同分析器索引的相同内容进行操作。

> **注意**：在使用字段遮罩时，相关性分数是根据遮罩字段的特性（范数）计算的，而不是实际搜索的字段。这意味着如果遮罩字段与被搜索字段具有不同的属性（如长度或提升值），您可能会收到意外的评分结果。

## 相关指南（先读这些）

- [Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}})
- [多字段搜索]({{< relref "/docs/features/fulltext-search/multi-field-search.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

以下查询在词干化字段中搜索单词“long”，并查找“sleeve”一词的变体附近：

```
GET /clothing/_search
{
  "query": {
    "span_near": {
      "clauses": [
        {
          "span_term": {
            "description": "long"
          }
        },
        {
          "field_masking_span": {
            "query": {
              "span_term": {
                "description.stemmed": "sleev"
              }
            },
            "field": "description"
          }
        }
      ],
      "slop": 1,
      "in_order": true
    }
  }
}
```

查询匹配文档 1 和文档 4：

- 词项“long”在两个文档的 `description` 字段中都出现。
- 文档 1 包含单词“sleeved”，文档 4 包含单词“sleeves”。
- `field_masking_span` 使得词干化字段的匹配看起来像是出现在原始字段中。
- 这些词项在指定顺序中彼此相距 1 个位置（"long"必须出现在"sleeve"之前）。

```
{
  "took": 7,
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
    "max_score": 0.7444251,
    "hits": [
      {
        "_index": "clothing",
        "_id": "1",
        "_score": 0.7444251,
        "_source": {
          "description": "Long-sleeved dress shirt with a formal collar and button cuffs. "
        }
      },
      {
        "_index": "clothing",
        "_id": "4",
        "_score": 0.4291246,
        "_source": {
          "description": "A set of two midi silk shirt dresses with long fluttered sleeves in black. "
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 `field_masking_span` 查询支持的所有顶层参数。所有参数都是必需的。

| 参数    | 数据类型 | 描述                                                                     |
| ------- | -------- | ------------------------------------------------------------------------ |
| `query` | Object   | 要在实际字段上执行的 span 查询。                                         |
| `field` | String   | 用于遮蔽查询的字段名。其他 span 查询会把这个查询当作是在这个字段上执行。 |

