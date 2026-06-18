---
title: "Span Term 查询"
date: 0001-01-01
summary: "Span Term 查询 #  span_term 查询是最基本的 span 查询，它匹配包含单个词的 span。它是更复杂的 span 查询的构建模块。
例如，您可以使用 span_term 查询来：
 查找可用于其他 span 查询的精确词匹配。 匹配特定单词同时保持位置信息。 创建可与其他 span 查询组合的基本 span。  相关指南（先读这些） #    Span 查询  查询 DSL 基础  参考样例 #  以下查询搜索确切的词“formal”：
GET /clothing/_search { &#34;query&#34;: { &#34;span_term&#34;: { &#34;description&#34;: &#34;formal&#34; } } } 或者，您可以在 value 参数中指定搜索词：
GET /clothing/_search { &#34;query&#34;: { &#34;span_term&#34;: { &#34;description&#34;: { &#34;value&#34;: &#34;formal&#34; } } } } 您也可以指定 boost 值来提升文档得分："
---


# Span Term 查询

`span_term` 查询是最基本的 span 查询，它匹配包含单个词的 span。它是更复杂的 span 查询的构建模块。

例如，您可以使用 `span_term` 查询来：

- 查找可用于其他 span 查询的精确词匹配。
- 匹配特定单词同时保持位置信息。
- 创建可与其他 span 查询组合的基本 span。

## 相关指南（先读这些）

- [Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

## 参考样例

以下查询搜索确切的词“formal”：

```
GET /clothing/_search
{
  "query": {
    "span_term": {
      "description": "formal"
    }
  }
}
```

或者，您可以在 `value` 参数中指定搜索词：

```
GET /clothing/_search
{
  "query": {
    "span_term": {
      "description": {
        "value": "formal"
      }
    }
  }
}
```

您也可以指定 `boost` 值来提升文档得分：

```
GET /clothing/_search
{
  "query": {
    "span_term": {
      "description": {
        "value": "formal",
        "boost": 2
      }
    }
  }
}
```

该查询匹配文档 1 和文档 2，因为它们包含确切的词项“formal”。位置信息被保留以用于其他 span 查询。

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
    "max_score": 1.498922,
    "hits": [
      {
        "_index": "clothing",
        "_id": "2",
        "_score": 1.498922,
        "_source": {
          "description": "Beautiful long dress in red silk, perfect for formal events."
        }
      },
      {
        "_index": "clothing",
        "_id": "1",
        "_score": 1.4466847,
        "_source": {
          "description": "Long-sleeved dress shirt with a formal collar and button cuffs. "
        }
      }
    ]
  }
}
```

## 参数说明

下表列出了 `span_term` 查询支持的所有顶层参数。

| 参数      | 数据类型     | 描述                 |
| --------- | ------------ | -------------------- |
| `<field>` | 字符串或对象 | 要搜索的字段的名称。 |

