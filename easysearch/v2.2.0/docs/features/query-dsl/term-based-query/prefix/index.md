---
title: "Prefix 查询"
date: 0001-01-01
summary: "Prefix 查询 #  使用 prefix 查询可以搜索以特定前缀开头的词。例如，以下查询会搜索 speaker 字段包含以 KING H 开头的词的文档。
相关指南（先读这些） #    部分匹配  结构化搜索  GET shakespeare/_search { &#34;query&#34;: { &#34;prefix&#34;: { &#34;speaker&#34;: &#34;KING H&#34; } } } 为了提供参数，您可以使用与前面的查询相同的形式，并使用以下扩展语法
GET shakespeare/_search { &#34;query&#34;: { &#34;prefix&#34;: { &#34;speaker&#34;: { &#34;value&#34;: &#34;KING H&#34; } } } } 参数说明 #  查询接受字段名称（ &lt;field&gt; ）作为顶级参数：
GET _search { &#34;query&#34;: { &#34;prefix&#34;: { &#34;&lt;field&gt;&#34;: { &#34;value&#34;: &#34;sample&#34;, ... } } } } &lt;field&gt; 接受以下参数。除了 value 之外，所有参数都是可选的。"
---


# Prefix 查询

使用 `prefix` 查询可以搜索以特定前缀开头的词。例如，以下查询会搜索 `speaker` 字段包含以 `KING H` 开头的词的文档。

## 相关指南（先读这些）

- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

```
GET shakespeare/_search
{
  "query": {
    "prefix": {
      "speaker": "KING H"
    }
  }
}

```

为了提供参数，您可以使用与前面的查询相同的形式，并使用以下扩展语法

```
GET shakespeare/_search
{
  "query": {
    "prefix": {
      "speaker": {
        "value": "KING H"
      }
    }
  }
}

```

## 参数说明

查询接受字段名称（ `<field>` ）作为顶级参数：

```
GET _search
{
  "query": {
    "prefix": {
      "<field>": {
        "value": "sample",
        ...
      }
    }
  }
}

```

`<field>` 接受以下参数。除了 `value` 之外，所有参数都是可选的。

| 参数               | 数据类型 | 描述                                                                                                                                                                                                                     |
| ------------------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `value`            | String   | 在由 <field> 指定的字段中搜索的词项。                                                                                                                                                                                    |
| `boost`            | Float    | 一个浮点值，用于指定该字段对相关性评分的权重。值大于 1.0 会增加字段的相关性。值在 0.0 到 1.0 之间会降低字段的相关性。默认值为 1.0。                                                                                      |
| `case_insensitive` | Boolean  | 如果设置为 true ，则允许对字段索引值进行不区分大小写的匹配。默认值为 false （大小写敏感性由字段的映射决定）。                                                                                                            |
| `rewrite`          | String   | 决定 Easysearch 如何重写和评分多词查询。有效值为 `constant_score` 、 `scoring_boolean` 、 `constant_score_boolean` 、 `top_terms_N` 、 `top_terms_boost_N` 和 `top_terms_blended_freqs_N` 。默认值为 `constant_score` 。 |

> 如果将 search.allow_expensive_queries 设置为 false ，则不会执行前缀查询。如果启用了 index_prefixes ，则会忽略 search.allow_expensive_queries 设置，并构建并运行一个优化后的查询。

## 工作原理

`prefix` 查询是一个 **词级别（term-level）** 的底层查询，它不会在搜索之前分析查询字符串——传入的前缀就是要查找的前缀。

其执行过程如下：

1. 扫描倒排索引的有序词列表，找到第一个以指定前缀开头的词
2. 收集该词关联的文档 ID
3. 移动到下一个词，如果也以该前缀开头则重复第 2 步
4. 持续直到遇到不以该前缀开头的词

例如，倒排索引如下：

```
Term:          Doc IDs:
-------------------------
"SW5 0BE"    |  5
"W1F 7HW"    |  3
"W1V 3DG"    |  1
"W2F 8HW"    |  2
"WC1N 1LZ"   |  4
-------------------------
```

搜索前缀 `W1` 时，Easysearch 会依次匹配到 `W1F 7HW` 和 `W1V 3DG`，返回文档 3 和 1。

## 性能与优化

> **注意**：前缀越短，需要访问的词项越多。以 `W` 作为前缀可能需要扫描数百万个词项，对集群产生很大压力。

### 使用 index_prefixes 优化

在映射中对字段启用 `index_prefixes`，可以让 Easysearch 在索引阶段预先生成前缀的子字段，从而大幅提升前缀查询的性能：

```
PUT my_index
{
  "mappings": {
    "properties": {
      "product_name": {
        "type": "text",
        "index_prefixes": {
          "min_chars": 2,
          "max_chars": 5
        }
      }
    }
  }
}
```

### 在分析过的字段上使用时

`prefix` 查询作用于倒排索引中的 **单个词项**，而不是原始字段值。如果对一个 `text` 字段使用 `prefix` 查询，它会匹配分词后的每个词项的前缀：

- 字段值 `"Quick brown fox"` → 词项 `quick`、`brown`、`fox`
- `prefix: "bro"` → ✅ 匹配（匹配词项 `brown`）
- `prefix: "Qui"` → ❌ 不匹配（索引中的词项是小写的 `quick`）
- `prefix: "quick bro"` → ❌ 不匹配（前缀不会跨词项匹配）

如果需要对精确值做前缀匹配，应使用 `keyword` 类型的字段。

