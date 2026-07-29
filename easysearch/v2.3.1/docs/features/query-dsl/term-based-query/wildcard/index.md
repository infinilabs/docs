---
title: "Wildcard 查询"
date: 0001-01-01
summary: "Wildcard 查询 #  使用 wildcard 查询来搜索匹配通配符模式的词项。通配符查询支持以下操作符。
相关指南（先读这些） #    部分匹配  结构化搜索  通配符字段类型（Wildcard） — 专为高效通配符匹配设计的字段类型     操作符 描述     * 匹配零个或多个字符。   ? 匹配任意单个字符。   case_insensitive 若 true 为真，则通配符查询不区分大小写；若 false 为真，则通配符查询区分大小写。默认情况下 false 为真（区分大小写）。    若进行区分大小写的搜索，查找以 H 开头且以 Y 结尾的词，可使用以下请求：
GET shakespeare/_search { &#34;query&#34;: { &#34;wildcard&#34;: { &#34;speaker&#34;: { &#34;value&#34;: &#34;H*Y&#34;, &#34;case_insensitive&#34;: false } } } } 如果你将 * 更改为 ?"
---


# Wildcard 查询

使用 `wildcard` 查询来搜索匹配通配符模式的词项。通配符查询支持以下操作符。

## 相关指南（先读这些）

- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})
- [通配符字段类型（Wildcard）]({{< relref "/docs/features/mapping-and-analysis/field-types/wildcard.md" >}}) — 专为高效通配符匹配设计的字段类型

| 操作符             | 描述                                                                                                                       |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| `*`                | 匹配零个或多个字符。                                                                                                       |
| `?`                | 匹配任意单个字符。                                                                                                         |
| `case_insensitive` | 若 `true` 为真，则通配符查询不区分大小写；若 `false` 为真，则通配符查询区分大小写。默认情况下 `false` 为真（区分大小写）。 |

若进行区分大小写的搜索，查找以 `H` 开头且以 `Y` 结尾的词，可使用以下请求：

```

GET shakespeare/_search
{
  "query": {
    "wildcard": {
      "speaker": {
        "value": "H*Y",
        "case_insensitive": false
      }
    }
  }
}

```

如果你将 `*` 更改为 `?` ，则不会有任何匹配，因为 `?` 引用的是一个单一字符。

> **性能注意**：通配符查询通常速度较慢，因为它们需要遍历大量的词项。避免在查询的开头使用通配符字符，因为这在资源和时间方面都可能是一项非常昂贵的操作。更多性能优化建议，请参考[部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})章节。

## 参数说明

查询接受字段名称（ `<field>` ）作为顶级参数：

```
GET _search
{
  "query": {
    "wildcard": {
      "<field>": {
        "value": "patt*rn",
        ...
      }
    }
  }
}

```

`<field>` 接受以下参数。除了 `value` 之外，所有参数都是可选的。

| 参数               | 数据类型 | 描述                                                                                                                                                                                                                     |
| ------------------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `value`            | String   | 用于匹配 `<field>` 中指定字段的词条的通配符模式。                                                                                                                                                                        |
| `boost`            | Float    | 一个浮点数，用于指定该字段对相关性评分的权重。值大于 1.0 会增加字段的相关性。值在 0.0 到 1.0 之间会降低字段的相关性。默认值为 1.0。                                                                                      |
| `case_insensitive` | Boolean  | 如果为 true，则允许不区分大小写地匹配值与索引字段值。默认值为 false （大小写敏感性由字段的映射决定）。                                                                                                                   |
| `rewrite`          | String   | 确定 Easysearch 如何重写和评分多词查询。有效值为 `constant_score` 、 `scoring_boolean` 、 `constant_score_boolean` 、 `top_terms_N` 、 `top_terms_boost_N` 和 `top_terms_blended_freqs_N` 。默认值为 `constant_score` 。 |

> 如果 search.allow_expensive_queries 设置为 false ，则不会执行通配符查询。

## 工作原理

`wildcard` 查询和 `prefix`、`regexp` 查询一样，是词级别的底层查询。它需要扫描倒排索引中的词列表来查找所有匹配的词项，然后收集每个词项对应的文档 ID。

因此，以下原则同样适用：

- 通配符匹配的是倒排索引中的 **单个词项**，而不是原始字段值
- 对 `text` 类型字段使用时，匹配的是分词后的词项（例如小写化后的词）
- 对精确匹配原始值，应使用 `keyword` 类型字段

## 更多示例

### 匹配文件扩展名

```
GET files/_search
{
  "query": {
    "wildcard": {
      "filename.keyword": {
        "value": "*.pdf"
      }
    }
  }
}
```

### 匹配特定格式的编码

使用 `?` 匹配固定长度的模式。例如匹配 `A` 开头、后跟两个任意字符、以 `00` 结尾的产品编码：

```
GET products/_search
{
  "query": {
    "wildcard": {
      "sku.keyword": {
        "value": "A??00"
      }
    }
  }
}
```

### 不区分大小写的匹配

```
GET logs/_search
{
  "query": {
    "wildcard": {
      "message.keyword": {
        "value": "*error*",
        "case_insensitive": true
      }
    }
  }
}
```

## 性能优化建议

| 模式             | 性能 | 说明                                   |
| ---------------- | ---- | -------------------------------------- |
| `abc*`           | ✅ 好 | 等同于 prefix 查询，可快速定位          |
| `a*z`            | ⚠️ 中 | 先找前缀 `a`，再逐项过滤               |
| `*abc`           | ❌ 差 | 左通配符，必须扫描所有词项              |
| `*abc*`          | ❌ 差 | 双通配符，全量扫描                      |

> **建议**：对于左通配符（`*keyword`）这类查询，可考虑使用 `keyword` 字段配合自定义的 **reverse token filter** 在索引时将词项反转存储，然后用 prefix 查询替代。

## 相关查询对比

| 查询类型   | 语法灵活性 | 性能         | 适用场景                      |
| ---------- | ---------- | ------------ | ----------------------------- |
| `prefix`   | 低（仅前缀） | 较好       | 前缀匹配，如搜索建议          |
| `wildcard` | 中（`*` `?`） | 中等      | 简单的模式匹配                |
| `regexp`   | 高（完整正则） | 较差      | 复杂模式匹配                  |

> **提示**：如果需要频繁执行通配符查询（特别是左通配符 `*keyword`），建议使用 [通配符字段类型（Wildcard）]({{< relref "/docs/features/mapping-and-analysis/field-types/wildcard.md" >}}) 替代 `keyword` 字段。`wildcard` 字段类型通过 N-gram 子串索引优化了通配符和正则匹配的性能。
