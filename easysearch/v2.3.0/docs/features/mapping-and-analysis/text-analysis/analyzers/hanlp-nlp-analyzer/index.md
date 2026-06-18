---
title: "HanLP NLP 分析器（HanLP NLP）"
date: 0001-01-01
summary: "HanLP NLP 分析器 #  hanlp_nlp 分析器是为中文 NLP 分词的分析器，使用 HanLP NLP 模式，支持命名实体识别，适合需要高精度语义分析的场景。
需要安装 analysis-hanlp 插件。  分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 hanlp_nlp 分词器：使用 HanLP NLP 模式，支持命名实体识别 lowercase 分词过滤器：转换为小写  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;hanlp_nlp&#34;, &#34;text&#34;: &#34;刘德华和张学友是好朋友&#34; } 相关指南 #    文本分析基础  "
---


# HanLP NLP 分析器

`hanlp_nlp` 分析器是为中文 NLP 分词的分析器，使用 HanLP NLP 模式，支持命名实体识别，适合需要高精度语义分析的场景。

{{< hint info >}}
需要安装 `analysis-hanlp` 插件。
{{< /hint >}}

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `hanlp_nlp` 分词器：使用 HanLP NLP 模式，支持命名实体识别
- `lowercase` 分词过滤器：转换为小写

## 示例

```json
POST _analyze
{
  "analyzer": "hanlp_nlp",
  "text": "刘德华和张学友是好朋友"
}
```

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

