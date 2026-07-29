---
title: "规范化分词过滤器（Normalization）"
date: 0001-01-01
summary: "Normalization 分词过滤器 #  归一化分词过滤器旨在以减少文本差异（尤其是特殊字符差异）的方式对文本进行调整和简化。它主要用于通过对特定语言中的字符进行标准化处理，来应对书写上的差异。
相关指南（先读这些） #    文本分析：规范化  文本分析基础  以下是可用的归一化分词过滤器：
 阿拉伯语归一化: arabic_normalization 德语归一化: german_normalization 印地语归一化: hindi_normalization 印度语系归一化: indic_normalization 索拉尼语归一化: sorani_normalization 波斯语归一化: persian_normalization 斯堪的纳维亚语归一化: scandinavian_normalization 斯堪的纳维亚语折叠处理归一化: scandinavian_folding 塞尔维亚语归一化: serbian_normalization  参考样例 #  以下示例请求创建了一个名为 german_normalizer_example 的新索引，并配置了一个带有 german_normalization的分词器：
PUT /german_normalizer_example { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;german_normalizer&#34;: { &#34;type&#34;: &#34;german_normalization&#34; } }, &#34;analyzer&#34;: { &#34;german_normalizer_analyzer&#34;: { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;german_normalizer&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Normalization 分词过滤器

归一化分词过滤器旨在以减少文本差异（尤其是特殊字符差异）的方式对文本进行调整和简化。它主要用于通过对特定语言中的字符进行标准化处理，来应对书写上的差异。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

以下是可用的归一化分词过滤器：

- 阿拉伯语归一化:[arabic_normalization](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/ar/ArabicNormalizer.html)
- 德语归一化:[german_normalization](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/de/GermanNormalizationFilter.html)
- 印地语归一化:[hindi_normalization](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/hi/HindiNormalizer.html)
- 印度语系归一化:[indic_normalization](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/in/IndicNormalizer.html)
- 索拉尼语归一化:[sorani_normalization](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/ckb/SoraniNormalizer.html)
- 波斯语归一化:[persian_normalization](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/fa/PersianNormalizer.html)
- 斯堪的纳维亚语归一化:[scandinavian_normalization](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianNormalizationFilter.html)
- 斯堪的纳维亚语折叠处理归一化:[scandinavian_folding](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianFoldingFilter.html)
- 塞尔维亚语归一化:[serbian_normalization](https://lucene.apache.org/core/8_7_0/analyzers-common/org/apache/lucene/analysis/sr/SerbianNormalizationFilter.html)

## 参考样例

以下示例请求创建了一个名为 `german_normalizer_example` 的新索引，并配置了一个带有 `german_normalization`的分词器：

```
PUT /german_normalizer_example
{
  "settings": {
    "analysis": {
      "filter": {
        "german_normalizer": {
          "type": "german_normalization"
        }
      },
      "analyzer": {
        "german_normalizer_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "german_normalizer"
          ]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /german_normalizer_example/_analyze
{
  "text": "Straße München",
  "analyzer": "german_normalizer_analyzer"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "strasse",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "munchen",
      "start_offset": 7,
      "end_offset": 14,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

