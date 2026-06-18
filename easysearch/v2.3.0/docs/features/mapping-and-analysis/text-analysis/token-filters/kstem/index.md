---
title: "KStem 词干分词过滤器（KStem）"
date: 0001-01-01
summary: "KStem 分词过滤器 #  kstem 分词过滤器是一种词干提取过滤器，用于将单词还原为其词根形式。该过滤器是一种专为英语设计的轻量级算法词干提取器，它执行以下词干提取操作：
 将复数形式的单词还原为单数形式。 将不同的动词时态转换为其基本形式。 去除常见的派生词尾，例如 &ldquo;-ing&rdquo; 或 &ldquo;-ed&rdquo;。  kstem 分词过滤器等同于配置了 light_english 语言的词干提取器过滤器。与其他词干提取过滤器（如 porter_stem）相比，它提供了更为保守的词干提取方式。
相关指南（先读这些） #    文本分析：词干提取  文本分析：规范化  KStem 分词过滤器基于 Lucene 的 KStemFilter。如需了解更多信息，请参阅 Lucene 文档。
参考样例 #  以下示例请求创建了一个名为 my_kstem_index 的新索引，并配置了一个带有 KStem 过滤器的分词器：
PUT /my_kstem_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;kstem_filter&#34;: { &#34;type&#34;: &#34;kstem&#34; } }, &#34;analyzer&#34;: { &#34;my_kstem_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;kstem_filter&#34; ] } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_kstem_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# KStem 分词过滤器

`kstem` 分词过滤器是一种词干提取过滤器，用于将单词还原为其词根形式。该过滤器是一种专为英语设计的轻量级算法词干提取器，它执行以下词干提取操作：

1. 将复数形式的单词还原为单数形式。
2. 将不同的动词时态转换为其基本形式。
3. 去除常见的派生词尾，例如 "-ing" 或 "-ed"。

`kstem` 分词过滤器等同于配置了 `light_english` 语言的词干提取器过滤器。与其他词干提取过滤器（如 `porter_stem`）相比，它提供了更为保守的词干提取方式。

## 相关指南（先读这些）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

KStem 分词过滤器基于 Lucene 的 `KStemFilter`。如需了解更多信息，请参阅 [Lucene 文档](https://lucene.apache.org/core/9_10_0/analysis/common/org/apache/lucene/analysis/en/KStemFilter.html)。

## 参考样例

以下示例请求创建了一个名为 `my_kstem_index` 的新索引，并配置了一个带有 KStem 过滤器的分词器：

```
PUT /my_kstem_index
{
  "settings": {
    "analysis": {
      "filter": {
        "kstem_filter": {
          "type": "kstem"
        }
      },
      "analyzer": {
        "my_kstem_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "kstem_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_kstem_analyzer"
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_kstem_index/_analyze
{
  "analyzer": "my_kstem_analyzer",
  "text": "stops stopped"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "stop",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "stop",
      "start_offset": 6,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

