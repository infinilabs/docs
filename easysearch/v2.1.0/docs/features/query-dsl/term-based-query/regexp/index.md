---
title: "Regexp 查询"
date: 0001-01-01
summary: "Regexp 查询 #  使用 regexp 查询来搜索符合正则表达式的词项。有关编写正则表达式的更多信息，请参见正则表达式语法。
相关指南（先读这些） #    部分匹配  结构化搜索  以下查询搜索以任何大写或小写字母开头的任何词项 amlet ：
GET shakespeare/_search { &#34;query&#34;: { &#34;regexp&#34;: { &#34;play_name&#34;: &#34;[a-zA-Z]amlet&#34; } } }  重要注意事项：
 正则表达式应用于字段中的词条（即，标记/token），而不是整个字段。 默认情况下，正则表达式的最大长度为 1,000 个字符。要更改最大长度，请更新 index.max_regex_length 设置。 正则表达式使用 Lucene 语法，这与更标准的实现有所不同。请充分测试以确保获得预期的结果。 为了提高正则表达式查询的性能，避免使用没有前缀或后缀的通配符模式，例如 .* 或 .*?+ 。 regexp 查询可能会非常耗时，并且需要将 search.allow_expensive_queries 设置为 true 。在频繁执行 regexp 查询之前，请测试其对集群性能的影响，并考虑使用其他可能达到类似效果的查询。更多性能优化建议，请参考 部分匹配章节。   参数说明 #  查询接受字段名称（ &lt;field&gt; ）作为顶级参数：
GET _search { &#34;query&#34;: { &#34;regexp&#34;: { &#34;&lt;field&gt;&#34;: { &#34;value&#34;: &#34;[Ss]ample&#34;, ."
---


# Regexp 查询

使用 `regexp` 查询来搜索符合正则表达式的词项。有关编写正则表达式的更多信息，请参见正则表达式语法。

## 相关指南（先读这些）

- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

以下查询搜索以任何大写或小写字母开头的任何词项 `amlet` ：

```
GET shakespeare/_search
{
  "query": {
    "regexp": {
      "play_name": "[a-zA-Z]amlet"
    }
  }
}

```

> **重要注意事项**：
> - 正则表达式应用于字段中的词条（即，标记/token），而不是整个字段。
> - 默认情况下，正则表达式的最大长度为 1,000 个字符。要更改最大长度，请更新 `index.max_regex_length` 设置。
> - 正则表达式使用 Lucene 语法，这与更标准的实现有所不同。请充分测试以确保获得预期的结果。
> - 为了提高正则表达式查询的性能，避免使用没有前缀或后缀的通配符模式，例如 `.*` 或 `.*?+` 。
> - `regexp` 查询可能会非常耗时，并且需要将 `search.allow_expensive_queries` 设置为 `true` 。在频繁执行 `regexp` 查询之前，请测试其对集群性能的影响，并考虑使用其他可能达到类似效果的查询。更多性能优化建议，请参考[部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})章节。

## 参数说明

查询接受字段名称（ `<field>` ）作为顶级参数：

```
GET _search
{
  "query": {
    "regexp": {
      "<field>": {
        "value": "[Ss]ample",
        ...
      }
    }
  }
}

```

`<field>` 接受以下参数。除了 `value` 之外，所有参数都是可选的。

| 参数                      | 数据类型 | 描述                                                                                                                                                                                                                     |
| ------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `value`                   | String   | 用于匹配指定在 `<field>` 字段中的项的正则表达式。                                                                                                                                                                        |
| `boost`                   | Float    | 一个浮点数，用于指定该字段对相关性评分的权重。值大于 1.0 会增加字段的相关性。值在 0.0 到 1.0 之间会降低字段的相关性。默认值为 1.0。                                                                                      |
| `case_insensitive`        | Boolean  | 如果 `true` ，则允许对正则表达式值与索引字段值进行不区分大小写的匹配。默认值为 `false` （大小写敏感性由字段的映射决定）。                                                                                                |
| `flags`                   | String   | 启用 Lucene 正则表达式引擎的可选操作符。有效值请参见可选操作符。                                                                                                                                                         |
| `flags_value`             | Integer  | `flags` 的整数替代形式，直接使用 Lucene 正则表达式 flag 的位掩码数值。与 `flags` 互斥，二者只需指定一个。                                                                                                               |
| `max_determinized_states` | Integer  | Lucene 将正则表达式转换为具有多个确定状态的自动机。此参数指定查询所需的自动机状态的最大数量。使用此参数以防止资源消耗过高。要运行复杂的正则表达式，您可能需要增加此参数的值。默认值为 10,000。                           |
| `rewrite`                 | String   | 确定 Easysearch 如何重写和评分多词查询。有效值为 `constant_score` 、 `scoring_boolean` 、 `constant_score_boolean` 、 `top_terms_N` 、 `top_terms_boost_N` 和 `top_terms_blended_freqs_N` 。默认值为 `constant_score` 。 |

> 如果 search.allow_expensive_queries 设置为 false ，则 regexp 查询将不被执行。

## 支持的正则语法

Easysearch 使用 **Lucene 正则表达式语法**，与标准正则语法（PCRE）有所不同。以下是常用元素速查：

| 语法       | 含义                                     | 示例                              |
| ---------- | ---------------------------------------- | --------------------------------- |
| `.`        | 匹配任意单个字符                         | `a.c` → `abc`、`adc`             |
| `+`        | 前一个字符出现一次或多次                 | `ab+c` → `abc`、`abbc`           |
| `*`        | 前一个字符出现零次或多次                 | `ab*c` → `ac`、`abc`、`abbc`     |
| `?`        | 前一个字符出现零次或一次                 | `ab?c` → `ac`、`abc`             |
| `[abc]`    | 字符类，匹配其中任意一个                 | `[aeiou]` → 任意元音             |
| `[a-z]`    | 字符范围                                 | `[0-9]` → 任意数字               |
| `(ab\|cd)` | 交替，匹配其中任意一个                   | `(cat\|dog)` → `cat` 或 `dog`    |
| `{n,m}`    | 重复 n 到 m 次（需启用 `INTERVAL` flag） | `a{2,4}` → `aa`、`aaa`、`aaaa`   |

> **注意**：Lucene 不支持 `\d`、`\w`、`\s` 等快捷方式。需要用 `[0-9]`、`[a-zA-Z0-9_]` 等字符类来代替。

## 在分析过的字段上使用

`regexp` 查询作用于倒排索引中的 **单个词项**，而不是原始字段值。例如：

- 字段值 `"Quick brown fox"` → 分词后为词项 `quick`、`brown`、`fox`
- `regexp: "br.*"` → ✅ 匹配（匹配词项 `brown`）
- `regexp: "Qu.*"` → ❌ 不匹配（词项是小写 `quick`，不是 `Quick`）
- `regexp: "quick br.*"` → ❌ 不匹配（正则表达式不会跨越多个词项）

因此，如果需要对原始完整字段值做正则匹配，应使用 `keyword` 类型的字段。

## 替代方案

对于常见的部分匹配需求，下面这些方案往往性能更优：

| 需求                     | 推荐方案                                                              |
| ------------------------ | --------------------------------------------------------------------- |
| 以某前缀开头             | [prefix 查询]({{< relref "./prefix.md" >}})，开销更低                |
| 包含任意通配符           | [wildcard 查询]({{< relref "./wildcard.md" >}})，语法更简洁          |
| 搜索即输入（typeahead）  | 使用 `index_prefixes` 或 edge n-gram 分词器                          |
| 复杂文本模式匹配         | 考虑在索引时使用自定义分析器进行预处理                                |

