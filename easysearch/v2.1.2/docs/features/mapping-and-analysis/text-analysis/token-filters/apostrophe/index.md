---
title: "撇号分词过滤器（Apostrophe）"
date: 0001-01-01
summary: "Apostrophe 分词过滤器 #  apostrophe 分词过滤器的主要功能是移除所有格形式的撇号以及跟在撇号后面的所有内容。这在分析那些大量使用撇号的语言的文本时非常有用，比如土耳其语。在土耳其语中，撇号用于将词根与后缀分隔开来，这些后缀包括所有格后缀、格标记以及其他语法词尾。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参考样例 #  以下示例请求创建了一个名为 custom_text_index 的新索引，在索引设置中配置了一个自定义分词器，并在映射中使用该分词器：
PUT /custom_text_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;custom_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;apostrophe&#34; ] } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;custom_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /custom_text_index/_analyze { &#34;analyzer&#34;: &#34;custom_analyzer&#34;, &#34;text&#34;: &#34;John&#39;s car is faster than Peter&#39;s bike&#34; } 返回内容包含产生的词元"
---


# Apostrophe 分词过滤器

`apostrophe` 分词过滤器的主要功能是移除所有格形式的撇号以及跟在撇号后面的所有内容。这在分析那些大量使用撇号的语言的文本时非常有用，比如土耳其语。在土耳其语中，撇号用于将词根与后缀分隔开来，这些后缀包括所有格后缀、格标记以及其他语法词尾。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下示例请求创建了一个名为 `custom_text_index` 的新索引，在索引设置中配置了一个自定义分词器，并在映射中使用该分词器：

```
PUT /custom_text_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "apostrophe"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "custom_analyzer"
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /custom_text_index/_analyze
{
  "analyzer": "custom_analyzer",
  "text": "John's car is faster than Peter's bike"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "john",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "car",
      "start_offset": 7,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "is",
      "start_offset": 11,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "faster",
      "start_offset": 14,
      "end_offset": 20,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "than",
      "start_offset": 21,
      "end_offset": 25,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "peter",
      "start_offset": 26,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "bike",
      "start_offset": 34,
      "end_offset": 38,
      "type": "<ALPHANUM>",
      "position": 6
    }
  ]
}
```

> 内置的省略符号分词过滤器并不适用于像法语这样的语言，在法语中撇号会出现在单词的开头。例如，`C'est l'amour de l'école` 这句话使用该过滤器分词后将会得到四个词元：“C”、“l”、“de” 和 “l”。

