---
title: "简单分析器（Simple）"
date: 0001-01-01
summary: "Simple 分析器 #  simple 分析器是一种非常基础的分析器，它会将文本中的非字母字符串拆分成词项，并将这些词项转换为小写形式。与 standard 分析器不同的是，simple 分析器将除字母字符之外的所有内容都视为分隔符，这意味着它不会把数字、标点符号或特殊字符识别为词元的一部分。
相关指南（先读这些） #    文本分析基础  文本分析：识别词元  参考样例 #  以下命令创建一个名为 my_simple_index 并使用简单分词器的索引：
PUT /my_simple_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;simple&#34; } } } } 配置自定义分词器 #  以下命令配置了一个索引，该索引带有一个自定义分词器，这个自定义分词器等同于添加了 html_strip 字符过滤器的简单分词器：
PUT /my_custom_simple_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;char_filter&#34;: { &#34;html_strip&#34;: { &#34;type&#34;: &#34;html_strip&#34; } }, &#34;tokenizer&#34;: { &#34;my_lowercase_tokenizer&#34;: { &#34;type&#34;: &#34;lowercase&#34; } }, &#34;analyzer&#34;: { &#34;my_custom_simple_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;char_filter&#34;: [&#34;html_strip&#34;], &#34;tokenizer&#34;: &#34;my_lowercase_tokenizer&#34;, &#34;filter&#34;: [&#34;lowercase&#34;] } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_custom_simple_analyzer&#34; } } } } 产生的词元 #  以下请求用来检查分词器生成的词元："
---


# Simple 分析器

`simple` 分析器是一种非常基础的分析器，它会将文本中的非字母字符串拆分成词项，并将这些词项转换为小写形式。与 `standard` 分析器不同的是，`simple` 分析器将除字母字符之外的所有内容都视为分隔符，这意味着它不会把数字、标点符号或特殊字符识别为词元的一部分。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下命令创建一个名为 `my_simple_index` 并使用简单分词器的索引：

```
PUT /my_simple_index
{
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "simple"
      }
    }
  }
}
```

## 配置自定义分词器

以下命令配置了一个索引，该索引带有一个自定义分词器，这个自定义分词器等同于添加了 `html_strip` 字符过滤器的简单分词器：

```
PUT /my_custom_simple_index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "html_strip": {
          "type": "html_strip"
        }
      },
      "tokenizer": {
        "my_lowercase_tokenizer": {
          "type": "lowercase"
        }
      },
      "analyzer": {
        "my_custom_simple_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "my_lowercase_tokenizer",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "my_custom_simple_analyzer"
      }
    }
  }
}
```

## 产生的词元

以下请求用来检查分词器生成的词元：

```
POST /my_custom_simple_index/_analyze
{
  "analyzer": "my_custom_simple_analyzer",
  "text": "<p>The slow turtle swims over to dogs &copy; 2024!</p>"
}
```

返回内容中包含了产生的词元

```
{
  "tokens": [
    {"token": "the","start_offset": 3,"end_offset": 6,"type": "word","position": 0},
    {"token": "slow","start_offset": 7,"end_offset": 11,"type": "word","position": 1},
    {"token": "turtle","start_offset": 12,"end_offset": 18,"type": "word","position": 2},
    {"token": "swims","start_offset": 19,"end_offset": 24,"type": "word","position": 3},
    {"token": "over","start_offset": 25,"end_offset": 29,"type": "word","position": 4},
    {"token": "to","start_offset": 30,"end_offset": 32,"type": "word","position": 5},
    {"token": "dogs","start_offset": 33,"end_offset": 37,"type": "word","position": 6}
  ]
}
```

