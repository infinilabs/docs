---
title: "多路复用分词过滤器（Multiplexer）"
date: 0001-01-01
summary: "Multiplexer 分词过滤器 #  multiplexer 分词过滤器允许你通过应用不同的过滤器来创建同一词元的多个版本。当你想要以多种方式分析同一个词元时，这非常有用。例如，你可能希望使用不同的词干提取、同义词或 n-gram 过滤器来分析一个词元，并将所有生成的词元一起使用。这个分词过滤器的工作方式是复制分词流，并对每个副本应用不同的过滤器。
 注意：multiplexer 分词过滤器会从分词流中移除重复的词元。
  限制：multiplexer 分词过滤器不支持 synonym 过滤器、synonym_graph 分词过滤器或 shingle 分词过滤器，因为这些过滤器不仅需要分析当前词元，还需要分析后续的词元，以便正确确定如何转换输入内容。
 相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参数说明 #  多路复用分词过滤器可以使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     filters 可选 字符串列表 要应用于分词流每个副本的分词过滤器的逗号分隔列表。默认值为空列表。   preserve_original 可选 布尔值 是否将原始词元保留为输出之一。默认值为 true。    参考样例 #  以下示例请求创建了一个名为 multiplexer_index 的新索引，并配置了一个带有多路复用过滤器的分词器：
PUT /multiplexer_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;english_stemmer&#34;: { &#34;type&#34;: &#34;stemmer&#34;, &#34;name&#34;: &#34;english&#34; }, &#34;synonym_filter&#34;: { &#34;type&#34;: &#34;synonym&#34;, &#34;synonyms&#34;: [ &#34;quick,fast&#34; ] }, &#34;multiplexer_filter&#34;: { &#34;type&#34;: &#34;multiplexer&#34;, &#34;filters&#34;: [&#34;english_stemmer&#34;, &#34;synonym_filter&#34;], &#34;preserve_original&#34;: true } }, &#34;analyzer&#34;: { &#34;multiplexer_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;multiplexer_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Multiplexer 分词过滤器

`multiplexer` 分词过滤器允许你通过应用不同的过滤器来创建同一词元的多个版本。当你想要以多种方式分析同一个词元时，这非常有用。例如，你可能希望使用不同的词干提取、同义词或 n-gram 过滤器来分析一个词元，并将所有生成的词元一起使用。这个分词过滤器的工作方式是复制分词流，并对每个副本应用不同的过滤器。

> **注意**：`multiplexer` 分词过滤器会从分词流中移除重复的词元。

> **限制**：`multiplexer` 分词过滤器不支持 `synonym` 过滤器、`synonym_graph` 分词过滤器或 `shingle` 分词过滤器，因为这些过滤器不仅需要分析当前词元，还需要分析后续的词元，以便正确确定如何转换输入内容。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

多路复用分词过滤器可以使用以下参数进行配置。

| 参数                | 必需/可选 | 数据类型   | 描述                                                               |
| ------------------- | --------- | ---------- | ------------------------------------------------------------------ |
| `filters`           | 可选      | 字符串列表 | 要应用于分词流每个副本的分词过滤器的逗号分隔列表。默认值为空列表。 |
| `preserve_original` | 可选      | 布尔值     | 是否将原始词元保留为输出之一。默认值为 `true`。                    |

## 参考样例

以下示例请求创建了一个名为 `multiplexer_index` 的新索引，并配置了一个带有多路复用过滤器的分词器：

```
PUT /multiplexer_index
{
  "settings": {
    "analysis": {
      "filter": {
        "english_stemmer": {
          "type": "stemmer",
          "name": "english"
        },
        "synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "quick,fast"
          ]
        },
        "multiplexer_filter": {
          "type": "multiplexer",
          "filters": ["english_stemmer", "synonym_filter"],
          "preserve_original": true
        }
      },
      "analyzer": {
        "multiplexer_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "multiplexer_filter"
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
POST /multiplexer_index/_analyze
{
  "analyzer": "multiplexer_analyzer",
  "text": "The slow turtle hides from the quick dog"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "The",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "slow",
      "start_offset": 4,
      "end_offset": 8,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "turtle",
      "start_offset": 9,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "turtl",
      "start_offset": 9,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "hides",
      "start_offset": 16,
      "end_offset": 21,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "hide",
      "start_offset": 16,
      "end_offset": 21,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "from",
      "start_offset": 22,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "the",
      "start_offset": 27,
      "end_offset": 30,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "quick",
      "start_offset": 31,
      "end_offset": 36,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "fast",
      "start_offset": 31,
      "end_offset": 36,
      "type": "SYNONYM",
      "position": 6
    },
    {
      "token": "dog",
      "start_offset": 37,
      "end_offset": 40,
      "type": "<ALPHANUM>",
      "position": 7
    }
  ]
}
```

