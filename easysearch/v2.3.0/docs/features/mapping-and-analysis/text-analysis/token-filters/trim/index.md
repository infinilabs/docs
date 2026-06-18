---
title: "修剪分词过滤器（Trim）"
date: 0001-01-01
summary: "Trim 分词过滤器 #  trim 分词过滤器会从词元中去除前导和尾随的空白字符。
 注意：许多常用的分词器，例如 standard 分词器、keyword 分词器和 whitespace 分词器，在分词过程中会自动去除前导和尾随的空白字符。当使用这些分词器时，无需额外配置 trim 分词过滤器。
 相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参考样例 #  以下示例请求创建了一个名为 my_pattern_trim_index 的新索引，并配置了一个带有修剪过滤器和匹配词元生成器的分词器。其中，匹配词元生成器不会去除词元的前导和尾随空白字符。
PUT /my_pattern_trim_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_trim_filter&#34;: { &#34;type&#34;: &#34;trim&#34; } }, &#34;tokenizer&#34;: { &#34;my_pattern_tokenizer&#34;: { &#34;type&#34;: &#34;pattern&#34;, &#34;pattern&#34;: &#34;,&#34; } }, &#34;analyzer&#34;: { &#34;my_pattern_trim_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;my_pattern_tokenizer&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;my_trim_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Trim 分词过滤器

`trim` 分词过滤器会从词元中去除前导和尾随的空白字符。

> **注意**：许多常用的分词器，例如 `standard` 分词器、`keyword` 分词器和 `whitespace` 分词器，在分词过程中会自动去除前导和尾随的空白字符。当使用这些分词器时，无需额外配置 `trim` 分词过滤器。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下示例请求创建了一个名为 `my_pattern_trim_index` 的新索引，并配置了一个带有修剪过滤器和匹配词元生成器的分词器。其中，匹配词元生成器不会去除词元的前导和尾随空白字符。

```
PUT /my_pattern_trim_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_trim_filter": {
          "type": "trim"
        }
      },
      "tokenizer": {
        "my_pattern_tokenizer": {
          "type": "pattern",
          "pattern": ","
        }
      },
      "analyzer": {
        "my_pattern_trim_analyzer": {
          "type": "custom",
          "tokenizer": "my_pattern_tokenizer",
          "filter": [
            "lowercase",
            "my_trim_filter"
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
GET /my_pattern_trim_index/_analyze
{
  "analyzer": "my_pattern_trim_analyzer",
  "text": " Easysearch ,  is ,   powerful  "
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "easysearch",
      "start_offset": 0,
      "end_offset": 12,
      "type": "word",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 13,
      "end_offset": 18,
      "type": "word",
      "position": 1
    },
    {
      "token": "powerful",
      "start_offset": 19,
      "end_offset": 32,
      "type": "word",
      "position": 2
    }
  ]
}
```

