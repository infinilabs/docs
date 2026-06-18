---
title: "Span Multi Term 查询"
date: 0001-01-01
summary: "Span Multi Term 查询 #  span_multi 查询允许您将多词查询（如 wildcard、fuzzy、prefix、range 或 regexp）包装为 span 查询。这使您能够在其他 span 查询中使用这些更灵活的匹配查询。
例如，您可以使用 span_multi 查询来：
 查找具有相同前缀的词语，并与其他词语靠近。 匹配跨度内单词的模糊变体。 在跨度查询中使用正则表达式。   注意：span_multi 查询可能匹配多个词。为了避免过度内存使用，您可以：
 为多词查询设置 rewrite 参数。 使用 top_terms_* 重写方法。 如果你仅使用 span_multi 进行 prefix 查询，请考虑为文本字段启用 index_prefixes 选项。这将自动将字段上的任何 prefix 查询重写为匹配索引前缀的单词查询。   相关指南（先读这些） #    Span 查询  部分匹配  查询 DSL 基础  参考样例 #  span_multi 查询使用以下语法来包装 prefix 查询：
&#34;span_multi&#34;: { &#34;match&#34;: { &#34;prefix&#34;: { &#34;description&#34;: { &#34;value&#34;: &#34;flutter&#34; } } } } 以下查询搜索以“dress”开头的单词，在彼此最多 5 个单词的距离内靠近任何形式的“sleeve”："
---


# Span Multi Term 查询

`span_multi` 查询允许您将多词查询（如 `wildcard`、`fuzzy`、`prefix`、`range` 或 `regexp`）包装为 `span` 查询。这使您能够在其他 `span` 查询中使用这些更灵活的匹配查询。

例如，您可以使用 `span_multi` 查询来：

- 查找具有相同前缀的词语，并与其他词语靠近。
- 匹配跨度内单词的模糊变体。
- 在跨度查询中使用正则表达式。

> **注意**：`span_multi` 查询可能匹配多个词。为了避免过度内存使用，您可以：
>
> - 为多词查询设置 `rewrite` 参数。
> - 使用 `top_terms_*` 重写方法。
> - 如果你仅使用 `span_multi` 进行 `prefix` 查询，请考虑为文本字段启用 `index_prefixes` 选项。这将自动将字段上的任何 `prefix` 查询重写为匹配索引前缀的单词查询。

## 相关指南（先读这些）

- [Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}})
- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

`span_multi` 查询使用以下语法来包装 `prefix` 查询：

```
"span_multi": {
  "match": {
    "prefix": {
      "description": {
        "value": "flutter"
      }
    }
  }
}
```

以下查询搜索以“dress”开头的单词，在彼此最多 5 个单词的距离内靠近任何形式的“sleeve”：

```
GET /clothing/_search
{
  "query": {
    "span_near": {
      "clauses": [
        {
          "span_multi": {
            "match": {
              "prefix": {
                "description": {
                  "value": "dress"
                }
              }
            }
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
      "slop": 5,
      "in_order": false
    }
  }
}
```

查询匹配文档 1（“长袖连衣裙……”）和文档 4（“……长飘袖连衣裙……”），因为“dress”和“long”在两个文档中出现的最大距离内都存在。

```
{
  "took": 5,
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
    "max_score": 1.7590723,
    "hits": [
      {
        "_index": "clothing",
        "_id": "1",
        "_score": 1.7590723,
        "_source": {
          "description": "Long-sleeved dress shirt with a formal collar and button cuffs. "
        }
      },
      {
        "_index": "clothing",
        "_id": "4",
        "_score": 0.84792376,
        "_source": {
          "description": "A set of two midi silk shirt dresses with long fluttered sleeves in black. "
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 `span_multi` 查询支持的所有顶层参数。所有参数都是必需的。

| 参数    | 数据类型 | 描述                                                                                     |
| ------- | -------- | ---------------------------------------------------------------------------------------- |
| `match` | Object   | 要包装的多项式查询（可以是 `prefix` 、 `wildcard` 、 `fuzzy` 、 `range` 或 `regexp` ）。 |

