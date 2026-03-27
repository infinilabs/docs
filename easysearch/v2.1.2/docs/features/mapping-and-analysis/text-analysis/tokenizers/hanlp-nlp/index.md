---
title: "HanLP NLP 分词器（HanLP NLP）"
date: 0001-01-01
summary: "HanLP NLP 分词器 #  hanlp_nlp 分词器使用 HanLP NLP 分词模式，支持命名实体识别，适合需要高精度语义分析的场景。
需要安装 analysis-hanlp 插件，并确保 perceptron CWS 模型可用。
示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;tokenizer&#34;: &#34;hanlp_nlp&#34; } } } } } 相关指南 #    HanLP 分词器  HanLP NLP 分析器  文本分析基础  "
---


# HanLP NLP 分词器

`hanlp_nlp` 分词器使用 HanLP NLP 分词模式，支持命名实体识别，适合需要高精度语义分析的场景。

需要安装 `analysis-hanlp` 插件，并确保 perceptron CWS 模型可用。

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "hanlp_nlp"
        }
      }
    }
  }
}
```

## 相关指南

- [HanLP 分词器]({{< relref "./hanlp" >}})
- [HanLP NLP 分析器]({{< relref "../analyzers/hanlp-nlp-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

