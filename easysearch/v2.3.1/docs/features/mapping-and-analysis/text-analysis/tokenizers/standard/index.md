---
title: "标准分词器（Standard）"
date: 0001-01-01
summary: "Standard 分词器 #  standard 分词器是 Easysearch 中的默认分词器。它基于单词边界，采用一种基于语法的方法对文本进行分词，这种方法能够识别字母、数字以及标点等其他字符。它具有高度的通用性，适用于多种语言，它使用了 Unicode 文本分割规则（ UAX#29）来将文本分割成词元。
相关指南（先读这些） #    词汇识别  文本分析基础  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个使用标准词元生成器的分词器：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_standard_analyzer&#34;: { &#34;type&#34;: &#34;standard&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_standard_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /my_index/_analyze { &#34;analyzer&#34;: &#34;my_standard_analyzer&#34;, &#34;text&#34;: &#34;Easysearch is powerful, fast, and scalable."
---


# Standard 分词器

`standard` 分词器是 Easysearch 中的默认分词器。它基于单词边界，采用一种基于语法的方法对文本进行分词，这种方法能够识别字母、数字以及标点等其他字符。它具有高度的通用性，适用于多种语言，它使用了 Unicode 文本分割规则（[UAX#29](https://unicode.org/reports/tr29/)）来将文本分割成词元。

## 相关指南（先读这些）

- [词汇识别]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个使用标准词元生成器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_standard_analyzer": {
          "type": "standard"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_standard_analyzer"
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index/_analyze
{
  "analyzer": "my_standard_analyzer",
  "text": "Easysearch is powerful, fast, and scalable."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "easysearch",
      "start_offset": 0,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 11,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "powerful",
      "start_offset": 14,
      "end_offset": 22,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "fast",
      "start_offset": 24,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "and",
      "start_offset": 30,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "scalable",
      "start_offset": 34,
      "end_offset": 42,
      "type": "<ALPHANUM>",
      "position": 5
    }
  ]
}
```

## 参数说明

标准词元生成器可以使用以下参数进行配置。

| 参数               | 必需/可选 | 数据类型 | 描述                                                                                                           |
| ------------------ | --------- | -------- | -------------------------------------------------------------------------------------------------------------- |
| `max_token_length` | 可选      | 整数     | 设置生成词元的最大长度。若超过该长度，词元将按照 `max_token_length` 所配置的长度拆分为多个词元。默认值为 255。 |

