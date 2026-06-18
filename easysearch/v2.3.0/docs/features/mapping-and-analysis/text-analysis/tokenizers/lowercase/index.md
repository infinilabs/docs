---
title: "小写分词器（Lowercase）"
date: 0001-01-01
summary: "Lowercase 分词器 #  lowercase 分词器会在空白处将文本拆分成词条，然后把所有词条转换为小写形式。从功能上来说，这与配置一个 letter 分词器并搭配一个 lowercase 词元过滤器的效果是一样的。不过，使用 lowercase 分词器效率更高，因为分词操作是在一步之内完成的。
相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  文本分析基础  参考样例 #  以下示例请求创建了一个名为 my-lowercase-index 的新索引，并配置了一个使用小写词元生成器的分词器：
PUT /my-lowercase-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_lowercase_tokenizer&#34;: { &#34;type&#34;: &#34;lowercase&#34; } }, &#34;analyzer&#34;: { &#34;my_lowercase_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;my_lowercase_tokenizer&#34; } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /my-lowercase-index/_analyze { &#34;analyzer&#34;: &#34;my_lowercase_analyzer&#34;, &#34;text&#34;: &#34;This is a Test."
---


# Lowercase 分词器

`lowercase` 分词器会在空白处将文本拆分成词条，然后把所有词条转换为小写形式。从功能上来说，这与配置一个 `letter` 分词器并搭配一个 `lowercase` 词元过滤器的效果是一样的。不过，使用 `lowercase` 分词器效率更高，因为分词操作是在一步之内完成的。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建了一个名为 `my-lowercase-index` 的新索引，并配置了一个使用小写词元生成器的分词器：

```
PUT /my-lowercase-index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_lowercase_tokenizer": {
          "type": "lowercase"
        }
      },
      "analyzer": {
        "my_lowercase_analyzer": {
          "type": "custom",
          "tokenizer": "my_lowercase_tokenizer"
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /my-lowercase-index/_analyze
{
  "analyzer": "my_lowercase_analyzer",
  "text": "This is a Test. Easysearch 123!"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "this",
      "start_offset": 0,
      "end_offset": 4,
      "type": "word",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 5,
      "end_offset": 7,
      "type": "word",
      "position": 1
    },
    {
      "token": "a",
      "start_offset": 8,
      "end_offset": 9,
      "type": "word",
      "position": 2
    },
    {
      "token": "test",
      "start_offset": 10,
      "end_offset": 14,
      "type": "word",
      "position": 3
    },
    {
      "token": "easysearch",
      "start_offset": 16,
      "end_offset": 26,
      "type": "word",
      "position": 4
    }
  ]
}
```

