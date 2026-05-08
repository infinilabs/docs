---
title: "词干提取（Stemming）"
date: 0001-01-01
description: "词干提取组件配置参考：算法词干提取器与字典词干提取器的使用方法。"
summary: "词干提取配置参考 #  本页提供词干提取（Stemming）相关组件的技术配置参考。关于词干提取的概念、策略选择和最佳实践，请先阅读指南。
相关指南（概念与策略） #    文本分析：词干提取 - 词干提取的收益、风险与控制方式  文本分析基础 - 分析器原理   概述 #  词干提取是将单词还原为其词根或基本形式（即词干）的过程。这项技术可确保在搜索操作中，单词的不同变体都能匹配到相应结果。例如，单词 &ldquo;running&rdquo;（跑步，现在分词形式）、&ldquo;runner&rdquo;（跑步者，名词形式）和 &ldquo;ran&rdquo;（跑步，过去式）都可以还原为词干 &ldquo;run&rdquo;（跑步，原形），这样一来，搜索这些词中的任何一个都能返回相关结果。
在自然语言中，由于动词变位、名词复数变化或词的派生等原因，单词常常以各种形式出现。词干提取在以下方面提升了搜索操作的效果：
 提高搜索召回率：通过将不同的单词形式匹配到同一个词干，词干提取增加了检索到的相关文档的数量 减小索引大小：仅存储单词的词干形式可以减少搜索索引的总体大小  词干提取是通过在分析器中使用词元过滤器来配置的。一个分析器包含以下组件：
 字符过滤器：在分词之前修改字符流 分词器：将文本拆分为词元（通常是单词） 词元过滤器：在分词之后修改词元，例如，应用词干提取操作  使用内置词元过滤器进行词干提取的示例 #  要实现词干提取，你可以配置一个内置的词元过滤器，比如 porter_stem 或 kstem 过滤器。
波特词干提取算法（ Porter stemming algorithm）是一种常用于英语的词干提取算法。
创建带有自定义分词器的索引 #  以下示例请求创建了一个名为 my_stemming_index 的新索引，并配置了一个使用 porter_stem 词元过滤器的分词器：
PUT /my_stemming_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_stemmer_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;porter_stem&#34; ] } } } } } 此配置包含以下内容："
---


# 词干提取配置参考

本页提供词干提取（Stemming）相关组件的技术配置参考。关于词干提取的概念、策略选择和最佳实践，请先阅读指南。

## 相关指南（概念与策略）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming" >}}) - 词干提取的收益、风险与控制方式
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}}) - 分析器原理

---

## 概述

词干提取是将单词还原为其词根或基本形式（即词干）的过程。这项技术可确保在搜索操作中，单词的不同变体都能匹配到相应结果。例如，单词 "running"（跑步，现在分词形式）、"runner"（跑步者，名词形式）和 "ran"（跑步，过去式）都可以还原为词干 "run"（跑步，原形），这样一来，搜索这些词中的任何一个都能返回相关结果。

在自然语言中，由于动词变位、名词复数变化或词的派生等原因，单词常常以各种形式出现。词干提取在以下方面提升了搜索操作的效果：

- **提高搜索召回率**：通过将不同的单词形式匹配到同一个词干，词干提取增加了检索到的相关文档的数量
- **减小索引大小**：仅存储单词的词干形式可以减少搜索索引的总体大小

词干提取是通过在分析器中使用词元过滤器来配置的。一个分析器包含以下组件：

- **字符过滤器**：在分词之前修改字符流
- **分词器**：将文本拆分为词元（通常是单词）
- **词元过滤器**：在分词之后修改词元，例如，应用词干提取操作

## 使用内置词元过滤器进行词干提取的示例

要实现词干提取，你可以配置一个内置的词元过滤器，比如 `porter_stem` 或 `kstem` 过滤器。

波特词干提取算法（[Porter stemming algorithm](https://snowballstem.org/algorithms/porter/stemmer.html)）是一种常用于英语的词干提取算法。

### 创建带有自定义分词器的索引

以下示例请求创建了一个名为 `my_stemming_index` 的新索引，并配置了一个使用 `porter_stem` 词元过滤器的分词器：

```
PUT /my_stemming_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_stemmer_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "porter_stem"
          ]
        }
      }
    }
  }
}
```

此配置包含以下内容：

1. **标准分词器**：根据单词边界将文本拆分为词项。
2. **小写字母过滤器**：将所有词元转换为小写形式。
3. **波特词干过滤器（porter_stem 过滤器）**：将单词还原为它们的词根形式。

### 测试分词器

为了检验词干提取的效果，使用之前配置好的自定义分词器来分析一段示例文本：

```
POST /my_stemming_index/_analyze
{
  "analyzer": "my_stemmer_analyzer",
  "text": "The runners are running swiftly."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "the",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "runner",
      "start_offset": 4,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "ar",
      "start_offset": 12,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "run",
      "start_offset": 16,
      "end_offset": 23,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "swiftli",
      "start_offset": 24,
      "end_offset": 31,
      "type": "<ALPHANUM>",
      "position": 4
    }
  ]
}
```

## 词干提取器的类别

你可以配置属于以下两类的词干提取器：

- **算法词干提取器**
- **字典词干提取器**

### 算法词干提取器

算法词干提取器运用预先设定的规则，系统地去除单词的词缀（前缀和后缀），将单词还原为词干。以下这些词元过滤器使用了算法词干提取器：

- **porter_stem**：应用波特词干提取算法，去除常见的后缀，把单词还原为词干。例如，“running”（跑步，现在分词）会变成“run”（跑步，原形）。
- **kstem**：这是一款专为英语设计的轻量级词干提取器，它将算法词干提取与内置字典相结合。它能把复数形式还原为单数形式，将动词的各种时态转换为原形，并去除常见的派生词尾。
- **stemmer**：为包括英语在内的多种语言提供算法词干提取功能，有像 light_english（轻型英语词干提取算法）、minimal_english（最简英语词干提取算法）和 porter2 等不同词干提取算法可供选择。
- **snowball**：应用雪球算法，为包括英语、法语、德语等在内的多种语言提供高效且准确的词干提取服务。

### 字典词干提取器

字典词干提取器依赖于庞大的字典，将单词映射到它们的词根形式，有效地对不规则单词进行词干提取。它们会在预先编译好的列表中查找每个单词，以找到其对应的词干。这种操作对资源的消耗更大，但对于不规则单词以及那些看似词干相似但含义差异很大的单词，往往能得出更好的词干提取结果。

字典词干提取器中最具代表性的例子是 `hunspell` 词元过滤器，它使用 Hunspell——一个在许多开源应用程序中都有使用的拼写检查引擎。

### 注意事项

在选择词干提取器时，请注意以下几点：

- 当处理速度和内存效率是优先考虑的因素，并且所处理的语言具有相对规则的词形变化模式时，算法词干提取器是合适的选择。
- 当处理不规则单词形式时的准确性至关重要，并且有足够的资源来支持增加的内存使用和处理时间时，字典词干提取器是理想的选择。

### 额外的词干提取配置

尽管 “organize”（组织）和 “organic”（有机的）有共同的语言词根，这会导致词干提取器将它们都处理成 “organ”（器官；机构），但它们在概念上的差异是很大的。在实际的搜索场景中，这种共同的词根可能会导致在搜索结果中返回不相关的匹配项。

你可以通过以下方法来应对这些挑战：

- **显式的词干提取覆盖**：不必仅仅依赖算法词干提取，你可以定义特定的词干提取规则。使用 [`stemmer_override`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/stemmer-override" >}})（词干提取覆盖），你可以确保 "organize" 保持不变，而 "organic" 被还原为 "organ"。这提供了对词项最终形式的精细控制。
- **关键词保留**：为了保持重要词项的完整性，你可以使用 [`keyword_marker`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/keyword-marker" >}})（关键词标记）词元过滤器。这个过滤器将特定的单词指定为关键词，防止后续的词干提取器过滤器对它们进行修改。在这个例子中，你可以将 "organize" 标记为关键词，确保它按照其原本的形式进行索引。
- **条件词干提取控制**：[`condition`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/condition" >}})（条件）词元过滤器使你能够建立规则来确定一个词项是否应该进行词干提取。这些规则可以基于各种标准，例如词项是否出现在预定义的列表中。
- **特定语言的词项排除**：对于内置的语言分词器，`stem_exclusion`（词干提取排除）参数提供了一种指定应免于词干提取的单词的方法。例如，你可以将 "organize" 添加到 `stem_exclusion` 列表中，防止分词器对其进行词干提取。这对于在特定语言中保留特定词项的独特含义可能很有用。

---

## 词干提取相关词元过滤器

### 算法词干提取器

| 过滤器 | 说明 |
|--------|------|
| [`stemmer`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/stemmer" >}}) | 通用词干提取器，支持多种语言和算法 |
| [`snowball`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/snowball" >}}) | 基于 Snowball 算法的词干提取器 |
| [`porter_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/porter-stem" >}}) | Porter 英语词干提取算法 |
| [`kstem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/kstem" >}}) | 英语轻量级词干提取器（算法+字典） |

### 语言专用词干提取器

| 过滤器 | 语言 |
|--------|------|
| [`arabic_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/arabic-stem" >}}) | 阿拉伯语 |
| [`brazilian_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/brazilian-stem" >}}) | 巴西葡萄牙语 |
| [`czech_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/czech-stem" >}}) | 捷克语 |
| [`dutch_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/dutch-stem" >}}) | 荷兰语 |
| [`french_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/french-stem" >}}) | 法语 |
| [`german_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/german-stem" >}}) | 德语 |
| [`romanian_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/romanian-stem" >}}) | 罗马尼亚语 |
| [`swedish_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/swedish-stem" >}}) | 瑞典语 |
| [`turkish_stem`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/turkish-stem" >}}) | 土耳其语 |

### 词干提取控制

| 过滤器 | 说明 |
|--------|------|
| [`stemmer_override`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/stemmer-override" >}}) | 自定义词干提取规则，覆盖默认行为 |
| [`keyword_marker`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/keyword-marker" >}}) | 将指定词标记为关键词，阻止词干提取 |
| [`keyword_repeat`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/keyword-repeat" >}}) | 同时保留原词和词干形式 |
| [`condition`]({{< relref "/docs/features/mapping-and-analysis/text-analysis/token-filters/condition" >}}) | 按条件决定是否应用词干提取 |

