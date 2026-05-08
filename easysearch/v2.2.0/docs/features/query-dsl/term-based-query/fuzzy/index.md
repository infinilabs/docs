---
title: "Fuzzy 查询"
date: 0001-01-01
summary: "Fuzzy 查询 #  fuzzy 查询用于搜索包含与搜索词相似的词条的文档，相似度在允许的最大 Damerau-Levenshtein 距离范围内。Damerau-Levenshtein 距离衡量将一个词条变为另一个词条所需的一字符变化的数量。这些变化包括：
相关指南（先读这些） #     文本分析：模糊匹配
   全文搜索
  Replacements: 替换，cat 变为 bat
  Insertions: 插入，cat 变为 cats
  Deletions: 删除，cat 变为 at
  Transpositions: 转换，cat 变为 act
  模糊查询会生成一个包含所有可能扩展的搜索词列表，这些扩展在 Damerau-Levenshtein 距离内。你可以在 max_expansions 字段中指定此类扩展的最大数量。查询然后会搜索匹配任何扩展的文档。如果你将 transpositions 参数设置为 false ，则搜索将使用经典的 Levenshtein 距离。
以下示例查询搜索发言者 HALET （误写为 HAMLET ）。未指定最大编辑距离，因此使用默认的 AUTO 编辑距离：
GET shakespeare/_search { &#34;query&#34;: { &#34;fuzzy&#34;: { &#34;speaker&#34;: { &#34;value&#34;: &#34;HALET&#34; } } } } 返回内容包含所有发言者为 HAMLET 的文档。"
---


# Fuzzy 查询

`fuzzy` 查询用于搜索包含与搜索词相似的词条的文档，相似度在允许的最大 `Damerau-Levenshtein` 距离范围内。`Damerau-Levenshtein` 距离衡量将一个词条变为另一个词条所需的一字符变化的数量。这些变化包括：

## 相关指南（先读这些）

- [文本分析：模糊匹配]({{< relref "/docs/features/mapping-and-analysis/text-analysis-fuzzy.md" >}})
- [全文搜索]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})

- Replacements: 替换，cat 变为 bat
- Insertions: 插入，cat 变为 cats
- Deletions: 删除，cat 变为 at
- Transpositions: 转换，cat 变为 act

模糊查询会生成一个包含所有可能扩展的搜索词列表，这些扩展在 Damerau-Levenshtein 距离内。你可以在 `max_expansions` 字段中指定此类扩展的最大数量。查询然后会搜索匹配任何扩展的文档。如果你将 `transpositions` 参数设置为 `false` ，则搜索将使用经典的 Levenshtein 距离。

以下示例查询搜索发言者 `HALET` （误写为 `HAMLET` ）。未指定最大编辑距离，因此使用默认的 `AUTO` 编辑距离：

```
GET shakespeare/_search
{
  "query": {
    "fuzzy": {
      "speaker": {
        "value": "HALET"
      }
    }
  }
}
```

返回内容包含所有发言者为 `HAMLET` 的文档。

以下示例查询使用高级参数搜索单词 `HALET` ：

```
GET shakespeare/_search
{
  "query": {
    "fuzzy": {
      "speaker": {
        "value": "HALET",
        "fuzziness": "2",
        "max_expansions": 40,
        "prefix_length": 0,
        "transpositions": true,
        "rewrite": "constant_score"
      }
    }
  }
}

```

## 参数说明

查询接受字段名称（ <field> ）作为顶级参数：

```
GET _search
{
  "query": {
    "fuzzy": {
      "<field>": {
        "value": "sample",
        ...
      }
    }
  }
}
```

`<field>` 接受以下参数。除了 `value` 之外，所有参数都是可选的。

| 参数             | 数据类型               | 描述                                                                                                                                                                                                                        |
| ---------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `value`          | String                 | 在 `<field>` 指定的字段中搜索的词项。                                                                                                                                                                                       |
| `boost`          | Float                  | 一个浮点数，用于指定该字段对相关性评分的权重。值大于 1.0 会增加字段的相关性。值在 0.0 到 1.0 之间会降低字段的相关性。默认值为 1.0。                                                                                         |
| `fuzziness`      | `AUTO` 、 `0` 或正整数 | 在确定一个词项是否匹配某个值时，将一个词变为另一个词所需的字符编辑次数（插入、删除、替换）。例如， `wined` 和 `wind` 之间的距离为 1。默认值 `AUTO` 会根据每个词项的长度选择一个值，并且对于大多数用例来说是一个不错的选择。 |
| `max_expansions` | Positive integer       | 查询可以扩展的最大项数。模糊查询会扩展到在指定距离 `fuzziness` 内的匹配项。然后 Easysearch 尝试匹配这些项。默认值为 `50` 。                                                                                                 |
| `prefix_length`  | Positive integer       | 不考虑在模糊匹配中计算的前导字符数。默认值为 0 。                                                                                                                                                                           |
| `transpositions` | Boolean                | 是否将相邻字符互换（如 `ab` → `ba`）计为单次编辑。`true` 使用 Damerau-Levenshtein 距离，`false` 使用经典 Levenshtein 距离。默认值为 `true`。                                                                                |
| `rewrite`        | String                 | 确定 Easysearch 如何重写和评分多词查询。有效值为 `constant_score` 、 `scoring_boolean` 、 `constant_score_boolean` 、 `top_terms_N` 、 `top_terms_boost_N` 和 `top_terms_blended_freqs_N` 。默认值为 `constant_score` 。    |

> 在 max_expansions 中指定较大的值可能会导致性能下降，特别是在 prefix_length 设置为 0 时，因为 Easysearch 尝试匹配的单词变体数量会很大。

> 如果将 search.allow_expensive_queries 设置为 false，则不会执行模糊查询。

