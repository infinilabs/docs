---
title: "Term 查询"
date: 0001-01-01
summary: "Term 查询 #  使用 term 查询在字段中搜索确切的词项。例如，以下查询搜索包含确切的行号的行：
相关指南（先读这些） #    结构化搜索  Query DSL 基础  GET shakespeare/_search { &#34;query&#34;: { &#34;term&#34;: { &#34;line_id&#34;: { &#34;value&#34;: &#34;61809&#34; } } } }  注意：term 查询仅匹配确切的词项，不会对查询文本进行分词。避免在 text 字段上使用 term 查询，应使用 keyword 字段或 match 查询。更多信息，请参阅 结构化搜索。
 您可以在 case_insensitive 参数中指定查询应不区分大小写：
GET shakespeare/_search { &#34;query&#34;: { &#34;term&#34;: { &#34;speaker&#34;: { &#34;value&#34;: &#34;HAMLET&#34;, &#34;case_insensitive&#34;: true } } } } 返回内容包含匹配的文档，无论大小写是否有差异：
&#34;hits&#34;: { &#34;total&#34;: { &#34;value&#34;: 1582, &#34;relation&#34;: &#34;eq&#34; }, &#34;max_score&#34;: 2, &#34;hits&#34;: [ { &#34;_index&#34;: &#34;shakespeare&#34;, &#34;_id&#34;: &#34;32700&#34;, &#34;_score&#34;: 2, &#34;_source&#34;: { &#34;type&#34;: &#34;line&#34;, &#34;line_id&#34;: 32701, &#34;play_name&#34;: &#34;Hamlet&#34;, &#34;speech_number&#34;: 9, &#34;line_number&#34;: &#34;1."
---


# Term 查询

使用 `term` 查询在字段中搜索确切的词项。例如，以下查询搜索包含确切的行号的行：

## 相关指南（先读这些）

- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

```
GET shakespeare/_search
{
  "query": {
    "term": {
      "line_id": {
        "value": "61809"
      }
    }
  }
}

```

> **注意**：`term` 查询仅匹配确切的词项，不会对查询文本进行分词。避免在 `text` 字段上使用 `term` 查询，应使用 `keyword` 字段或 `match` 查询。更多信息，请参阅[结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})。

您可以在 `case_insensitive` 参数中指定查询应不区分大小写：

```
GET shakespeare/_search
{
  "query": {
    "term": {
      "speaker": {
        "value": "HAMLET",
        "case_insensitive": true
      }
    }
  }
}
```

返回内容包含匹配的文档，无论大小写是否有差异：

```
"hits": {
  "total": {
    "value": 1582,
    "relation": "eq"
  },
  "max_score": 2,
  "hits": [
    {
      "_index": "shakespeare",
      "_id": "32700",
      "_score": 2,
      "_source": {
        "type": "line",
        "line_id": 32701,
        "play_name": "Hamlet",
        "speech_number": 9,
        "line_number": "1.2.66",
        "speaker": "HAMLET",
        "text_entry": "[Aside]  A little more than kin, and less than kind."
      }
    },
  ...
}
```

## 参数说明

查询接受字段名称（ `<field>` ）作为顶级参数：

```
GET _search
{
  "query": {
    "term": {
      "<field>": {
        "value": "sample",
        ...
      }
    }
  }
}

```

`<field>` 接受以下参数。除 `value` 外，所有参数都是可选的。

| 参数               | 数据类型 | 描述                                                                                                                                    |
| ------------------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `value`            | String   | 要在 `<field>` 中指定的字段中搜索的词项。只有当文档的字段值与词项完全匹配（包括正确的空格和大小写）时，该文档才会出现在结果中。         |
| `boost`            | Float    | 一个浮点数值，用于指定该字段对相关性分数的权重。大于 1.0 的值会增加字段的权重。介于 0.0 和 1.0 之间的值会降低字段的权重。默认值为 1.0。 |
| `_name`            | String   | 查询标签的查询名称。可选。                                                                                                              |
| `case_insensitive` | Boolean  | 如果为 `true` ，允许将值与索引字段的值进行不区分大小写的匹配。默认为 `false` （大小写敏感性由字段的映射决定）。                         |

