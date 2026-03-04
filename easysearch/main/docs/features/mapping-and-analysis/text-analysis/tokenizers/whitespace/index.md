---
title: "空白分词器（Whitespace）"
date: 0001-01-01
summary: "Whitespace 分词器 #  whitespace 分词器会依据空白字符（如空格、制表符和换行符）对文本进行拆分。它将由空白字符分隔开的每个单词都视为一个词元，并且不会执行诸如转换为小写形式或去除标点符号等额外的规范化操作。
相关指南（先读这些） #    文本分析：识别词元  文本分析基础  参考样例 #  以下示例请求创建一个名为 my_index 的新索引，并配置一个使用空格词元生成器的分词器：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;whitespace_tokenizer&#34;: { &#34;type&#34;: &#34;whitespace&#34; } }, &#34;analyzer&#34;: { &#34;my_whitespace_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;whitespace_tokenizer&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_whitespace_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /my_index/_analyze { &#34;analyzer&#34;: &#34;my_whitespace_analyzer&#34;, &#34;text&#34;: &#34;Easysearch is fast!"
---


# Whitespace 分词器

`whitespace` 分词器会依据空白字符（如空格、制表符和换行符）对文本进行拆分。它将由空白字符分隔开的每个单词都视为一个词元，并且不会执行诸如转换为小写形式或去除标点符号等额外的规范化操作。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建一个名为 `my_index` 的新索引，并配置一个使用空格词元生成器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "whitespace_tokenizer": {
          "type": "whitespace"
        }
      },
      "analyzer": {
        "my_whitespace_analyzer": {
          "type": "custom",
          "tokenizer": "whitespace_tokenizer"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_whitespace_analyzer"
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
  "analyzer": "my_whitespace_analyzer",
  "text": "Easysearch is fast! Really fast."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "Easysearch",
      "start_offset": 0,
      "end_offset": 10,
      "type": "word",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 11,
      "end_offset": 13,
      "type": "word",
      "position": 1
    },
    {
      "token": "fast!",
      "start_offset": 14,
      "end_offset": 19,
      "type": "word",
      "position": 2
    },
    {
      "token": "Really",
      "start_offset": 20,
      "end_offset": 26,
      "type": "word",
      "position": 3
    },
    {
      "token": "fast.",
      "start_offset": 27,
      "end_offset": 32,
      "type": "word",
      "position": 4
    }
  ]
}
```

## 参数说明

空格词元生成器可以使用以下参数进行配置。
| 参数 | 必需/可选 | 数据类型 | 描述 |
| --- | --- | --- | --- |
| `max_token_length` | 可选 | 整数 | 设置生成词元的最大长度。如果超过此长度，词元将按照 `max_token_length` 配置的长度拆分为多个词元。默认值为 255。 |

