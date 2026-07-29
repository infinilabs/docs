---
title: "N-gram 分词过滤器（N-gram）"
date: 0001-01-01
summary: "N-gram 分词过滤器 #  ngram 分词过滤器是一种强大的工具，用于将文本拆分为更小的组件，即 n-gram，这有助于提升部分匹配和模糊搜索能力。它通过将一个词元拆分成指定长度的子字符串来工作。这些过滤器在搜索应用程序中很常用，可用于支持自动补全、部分匹配以及容错拼写搜索。
相关指南（先读这些） #    部分匹配  文本分析：模糊匹配  文本分析：识别词元  参数说明 #  n-gram 分词过滤器可以使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     min_gram 可选 整数 n-gram 的最小长度。默认值为 1。   max_gram 可选 整数 n-gram 的最大长度。默认值为 2。   preserve_original 可选 布尔值 是否将原始词元保留为输出之一。默认值为 false。    参考样例 #  以下示例请求创建了一个名为“ngram_example_index”的新索引，并配置了一个带有 n-gram 过滤器的分词器：
PUT /ngram_example_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;ngram_filter&#34;: { &#34;type&#34;: &#34;ngram&#34;, &#34;min_gram&#34;: 2, &#34;max_gram&#34;: 3 } }, &#34;analyzer&#34;: { &#34;ngram_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;ngram_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# N-gram 分词过滤器

`ngram` 分词过滤器是一种强大的工具，用于将文本拆分为更小的组件，即 n-gram，这有助于提升部分匹配和模糊搜索能力。它通过将一个词元拆分成指定长度的子字符串来工作。这些过滤器在搜索应用程序中很常用，可用于支持自动补全、部分匹配以及容错拼写搜索。

## 相关指南（先读这些）

- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [文本分析：模糊匹配]({{< relref "/docs/features/mapping-and-analysis/text-analysis-fuzzy.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

n-gram 分词过滤器可以使用以下参数进行配置。

| 参数                | 必需/可选 | 数据类型 | 描述                                             |
| ------------------- | --------- | -------- | ------------------------------------------------ |
| `min_gram`          | 可选      | 整数     | n-gram 的最小长度。默认值为 1。                  |
| `max_gram`          | 可选      | 整数     | n-gram 的最大长度。默认值为 2。                  |
| `preserve_original` | 可选      | 布尔值   | 是否将原始词元保留为输出之一。默认值为 `false`。 |

## 参考样例

以下示例请求创建了一个名为“ngram_example_index”的新索引，并配置了一个带有 n-gram 过滤器的分词器：

```
PUT /ngram_example_index
{
  "settings": {
    "analysis": {
      "filter": {
        "ngram_filter": {
          "type": "ngram",
          "min_gram": 2,
          "max_gram": 3
        }
      },
      "analyzer": {
        "ngram_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "ngram_filter"
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
POST /ngram_example_index/_analyze
{
  "analyzer": "ngram_analyzer",
  "text": "Search"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "se",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "sea",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "ea",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "ear",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "ar",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "arc",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "rc",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "rch",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "ch",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    }
  ]
}
```

