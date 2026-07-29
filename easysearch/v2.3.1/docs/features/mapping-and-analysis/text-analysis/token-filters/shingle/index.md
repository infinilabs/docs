---
title: "词片分词过滤器（Shingle）"
date: 0001-01-01
summary: "Shingle 分词过滤器 #  shingle 分词过滤器用于从输入文本中生成词 n 元组（即词片）。例如，对于字符串 &ldquo;slow green turtle&rdquo;，词片过滤器会创建以下一元词片和二元词片：&ldquo;slow&rdquo;、&ldquo;slow green&rdquo;、&ldquo;green&rdquo;、&ldquo;green turtle&rdquo; 以及 &ldquo;turtle&rdquo;。
这个分词过滤器常常与其他过滤器结合使用，通过对短语而不是单个词元进行索引来提高搜索的准确性。
相关指南（先读这些） #    邻近匹配  文本分析：识别词元  文本分析：规范化  参数说明 #  词片分词过滤器可以使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     min_shingle_size 可选 整数 要连接的词元的最小数量。默认值为 2。   max_shingle_size 可选 整数 要连接的词元的最大数量。默认值为 2。   output_unigrams 可选 布尔值 是否将一元词（单个词元）包含在输出中。默认值为 true。   output_unigrams_if_no_shingles 可选 布尔值 如果未生成词片，是否输出一元词。默认值为 false。   token_separator 可选 字符串 用于将词元连接成词片的分隔符。默认值为一个空格（ ）。   filler_token 可选 字符串 插入到词元之间的空位置或间隙中的词元。默认值为下划线（_）。     如果 output_unigrams 和 output_unigrams_if_no_shingles 都设置为 true，则 output_unigrams_if_no_shingles 将被忽略。"
---


# Shingle 分词过滤器

`shingle` 分词过滤器用于从输入文本中生成词 n 元组（即词片）。例如，对于字符串 "slow green turtle"，词片过滤器会创建以下一元词片和二元词片："slow"、"slow green"、"green"、"green turtle" 以及 "turtle"。

这个分词过滤器常常与其他过滤器结合使用，通过对短语而不是单个词元进行索引来提高搜索的准确性。

## 相关指南（先读这些）

- [邻近匹配]({{< relref "/docs/features/fulltext-search/proximity-matching.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

词片分词过滤器可以使用以下参数进行配置。

| 参数                             | 必需/可选 | 数据类型 | 描述                                                          |
| -------------------------------- | --------- | -------- | ------------------------------------------------------------- |
| `min_shingle_size`               | 可选      | 整数     | 要连接的词元的最小数量。默认值为 2。                          |
| `max_shingle_size`               | 可选      | 整数     | 要连接的词元的最大数量。默认值为 2。                          |
| `output_unigrams`                | 可选      | 布尔值   | 是否将一元词（单个词元）包含在输出中。默认值为 true。         |
| `output_unigrams_if_no_shingles` | 可选      | 布尔值   | 如果未生成词片，是否输出一元词。默认值为 false。              |
| `token_separator`                | 可选      | 字符串   | 用于将词元连接成词片的分隔符。默认值为一个空格（` `）。       |
| `filler_token`                   | 可选      | 字符串   | 插入到词元之间的空位置或间隙中的词元。默认值为下划线（`_`）。 |

> 如果 `output_unigrams` 和 `output_unigrams_if_no_shingles` 都设置为 `true`，则 `output_unigrams_if_no_shingles` 将被忽略。

## 参数说明

以下示例请求创建了一个名为“my-shingle-index”的新索引，并配置了一个带有词片过滤器的分词器。

```
PUT /my-shingle-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_shingle_filter": {
          "type": "shingle",
          "min_shingle_size": 2,
          "max_shingle_size": 2,
          "output_unigrams": true
        }
      },
      "analyzer": {
        "my_shingle_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_shingle_filter"
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
GET /my-shingle-index/_analyze
{
  "analyzer": "my_shingle_analyzer",
  "text": "slow green turtle"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "slow",
      "start_offset": 0,
      "end_offset": 4,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "slow green",
      "start_offset": 0,
      "end_offset": 10,
      "type": "shingle",
      "position": 0,
      "positionLength": 2
    },
    {
      "token": "green",
      "start_offset": 5,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "green turtle",
      "start_offset": 5,
      "end_offset": 17,
      "type": "shingle",
      "position": 1,
      "positionLength": 2
    },
    {
      "token": "turtle",
      "start_offset": 11,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

