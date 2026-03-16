---
title: "ICU 分析器（ICU）"
date: 0001-01-01
summary: "ICU 分析器 #  icu 分析器是为多语言文本分析的分析器，基于 ICU（International Components for Unicode）实现，对亚洲语言混合文本提供比标准分析器更好的分词效果。
需要安装 analysis-icu 插件。  分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 icu_tokenizer 分词器：使用 ICU Unicode 文本分割算法 icu_normalizer 分词过滤器：Unicode 归一化（NFC 模式）  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;icu&#34;, &#34;text&#34;: &#34;Elasticsearch の全文検索エンジン&#34; } 相关指南 #    文本分析基础  "
---


# ICU 分析器

`icu` 分析器是为多语言文本分析的分析器，基于 ICU（International Components for Unicode）实现，对亚洲语言混合文本提供比标准分析器更好的分词效果。

{{< hint info >}}
需要安装 `analysis-icu` 插件。
{{< /hint >}}

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `icu_tokenizer` 分词器：使用 ICU Unicode 文本分割算法
- `icu_normalizer` 分词过滤器：Unicode 归一化（NFC 模式）

## 示例

```json
POST _analyze
{
  "analyzer": "icu",
  "text": "Elasticsearch の全文検索エンジン"
}
```

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

