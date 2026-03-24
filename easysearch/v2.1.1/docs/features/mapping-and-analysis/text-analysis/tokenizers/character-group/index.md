---
title: "字符组分词器（Character Group）"
date: 0001-01-01
summary: "Character Group 分词器 #  char_group 分词器使用特定字符作为分隔符将文本拆分为词元。它适用于需要简单直接进行分词的场景，为基于分词器的匹配模式提供了一种更简单的替代方案，避免了额外的复杂性。
相关指南（先读这些） #    文本分析：识别词元  文本分析基础  参考样例 #  以下示例请求创建了一个名为my_index的新索引，并配置了一个带有 char_group 字符组词元生成器的分词器。该词元生成器会依据空格、连字符 - 和冒号 : 来分割文本。
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_char_group_tokenizer&#34;: { &#34;type&#34;: &#34;char_group&#34;, &#34;tokenize_on_chars&#34;: [ &#34;whitespace&#34;, &#34;-&#34;, &#34;:&#34; ] } }, &#34;analyzer&#34;: { &#34;my_char_group_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;my_char_group_tokenizer&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_char_group_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Character Group 分词器

`char_group` 分词器使用特定字符作为分隔符将文本拆分为词元。它适用于需要简单直接进行分词的场景，为基于分词器的匹配模式提供了一种更简单的替代方案，避免了额外的复杂性。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建了一个名为`my_index`的新索引，并配置了一个带有 `char_group` 字符组词元生成器的分词器。该词元生成器会依据空格、连字符 `-` 和冒号 `:` 来分割文本。

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_char_group_tokenizer": {
          "type": "char_group",
          "tokenize_on_chars": [
            "whitespace",
            "-",
            ":"
          ]
        }
      },
      "analyzer": {
        "my_char_group_analyzer": {
          "type": "custom",
          "tokenizer": "my_char_group_tokenizer"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_char_group_analyzer"
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
  "analyzer": "my_char_group_analyzer",
  "text": "Fast-driving cars: they drive fast!"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "Fast",
      "start_offset": 0,
      "end_offset": 4,
      "type": "word",
      "position": 0
    },
    {
      "token": "driving",
      "start_offset": 5,
      "end_offset": 12,
      "type": "word",
      "position": 1
    },
    {
      "token": "cars",
      "start_offset": 13,
      "end_offset": 17,
      "type": "word",
      "position": 2
    },
    {
      "token": "they",
      "start_offset": 19,
      "end_offset": 23,
      "type": "word",
      "position": 3
    },
    {
      "token": "drive",
      "start_offset": 24,
      "end_offset": 29,
      "type": "word",
      "position": 4
    },
    {
      "token": "fast!",
      "start_offset": 30,
      "end_offset": 35,
      "type": "word",
      "position": 5
    }
  ]
}
```

## 参数说明

`char_group` 词元生成器可使用以下参数进行配置。

| 参数                | 必填/选填 | 数据类型 | 描述                                                                                                                                                                                                              |
| ------------------- | --------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tokenize_on_chars` | 必填      | 数组     | 指定用于对文本进行分词的一组字符。可以指定单个字符（例如 `-` 或 `@`），包括转义字符（例如 `\n` ），或者字符类，如空白字符（whitespace）、字母（letter）、数字（digit）、标点符号（punctuation）或符号（symbol）。 |
| `max_token_length`  | 选填      | 整数     | 设置生成词元的最大长度。如果超过此长度，词元将在`max_token_length`配置的长度处拆分为多个词元。默认值为 255。                                                                                                      |

