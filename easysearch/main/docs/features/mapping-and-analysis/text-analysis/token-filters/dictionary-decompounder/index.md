---
title: "字典复合词分词过滤器（Dictionary Decompounder）"
date: 0001-01-01
summary: "Dictionary Decompounder 分词过滤器 #  dictionary_decompounder 分词过滤器用于根据预定义的词典将复合词拆分为其组成部分。该过滤器对于德语、荷兰语或芬兰语等复合词较为常见的语言特别有用，因为将复合词拆分可以提高搜索的相关性。dictionary_decompounder 分词过滤器会根据已知词列表判断每个词元（单词）是否可以拆分为更小的词元。如果该词元可以拆分为已知单词，过滤器就会为该词元生成子词元。
相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参数说明 #  词典复合词分词过滤器具有以下参数：
   参数 必需/可选 数据类型 描述     word_list 除非配置了 word_list_path，否则为必需 字符串数组 过滤器用于拆分复合词的单词词典。   word_list_path 除非配置了 word_list，否则为必需 字符串 包含词典单词的文本文件的文件路径。可以接受绝对路径或相对于配置(config)目录的相对路径。词典文件必须采用 UTF-8 编码，且每个单词必须单独占一行。   min_word_size 可选 整数 会被考虑进行拆分的整个复合词的最小长度。如果复合词短于该值，则不会进行拆分。默认值为 5。   min_subword_size 可选 整数 任何子词的最小长度。如果子词短于该值，则不会包含在输出中。默认值为 2。   max_subword_size 可选 整数 任何子词的最大长度。如果子词长于该值，则不会包含在输出中。默认值为 15。   only_longest_match 可选 布尔值 如果设置为 true，则仅返回最长匹配的子词。默认值为 false。    参考样例 #  以下示例请求创建了一个名为 decompound_example 的新索引，并配置了一个带有词典复合词过滤器的分词器："
---


# Dictionary Decompounder 分词过滤器

`dictionary_decompounder` 分词过滤器用于根据预定义的词典将复合词拆分为其组成部分。该过滤器对于德语、荷兰语或芬兰语等复合词较为常见的语言特别有用，因为将复合词拆分可以提高搜索的相关性。`dictionary_decompounder` 分词过滤器会根据已知词列表判断每个词元（单词）是否可以拆分为更小的词元。如果该词元可以拆分为已知单词，过滤器就会为该词元生成子词元。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

词典复合词分词过滤器具有以下参数：

| 参数                 | 必需/可选                               | 数据类型   | 描述                                                                                                                                            |
| -------------------- | --------------------------------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `word_list`          | 除非配置了 `word_list_path`，否则为必需 | 字符串数组 | 过滤器用于拆分复合词的单词词典。                                                                                                                |
| `word_list_path`     | 除非配置了 `word_list`，否则为必需      | 字符串     | 包含词典单词的文本文件的文件路径。可以接受绝对路径或相对于配置(`config`)目录的相对路径。词典文件必须采用 UTF-8 编码，且每个单词必须单独占一行。 |
| `min_word_size`      | 可选                                    | 整数       | 会被考虑进行拆分的整个复合词的最小长度。如果复合词短于该值，则不会进行拆分。默认值为 `5`。                                                      |
| `min_subword_size`   | 可选                                    | 整数       | 任何子词的最小长度。如果子词短于该值，则不会包含在输出中。默认值为 `2`。                                                                        |
| `max_subword_size`   | 可选                                    | 整数       | 任何子词的最大长度。如果子词长于该值，则不会包含在输出中。默认值为 `15`。                                                                       |
| `only_longest_match` | 可选                                    | 布尔值     | 如果设置为 `true`，则仅返回最长匹配的子词。默认值为 `false`。                                                                                   |

## 参考样例

以下示例请求创建了一个名为 `decompound_example` 的新索引，并配置了一个带有词典复合词过滤器的分词器：

```
PUT /decompound_example
{
  "settings": {
    "analysis": {
      "filter": {
        "my_dictionary_decompounder": {
          "type": "dictionary_decompounder",
          "word_list": ["slow", "green", "turtle"]
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "my_dictionary_decompounder"]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /decompound_example/_analyze
{
  "analyzer": "my_analyzer",
  "text": "slowgreenturtleswim"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "slowgreenturtleswim",
      "start_offset": 0,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "slow",
      "start_offset": 0,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "green",
      "start_offset": 0,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "turtle",
      "start_offset": 0,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 0
    }
  ]
}
```

