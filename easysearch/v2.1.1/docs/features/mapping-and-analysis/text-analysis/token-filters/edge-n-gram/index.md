---
title: "边缘 N-gram 分词过滤器（Edge N-gram）"
date: 0001-01-01
summary: "Edge N-gram 分词过滤器 #  edge_ngram 分词过滤器与 ngram 分词过滤器非常相似，二者都会将特定字符串拆分成不同长度的子字符串。不过，edge_ngram 分词过滤器仅从词元的开头（边缘）生成 n 元子字符串。在自动补全或前缀匹配等场景中，它特别有用，因为在这些场景里，你希望在用户输入单词或短语时就能匹配词项的开头部分。
相关指南（先读这些） #    部分匹配  文本分析：识别词元  文本分析：规范化  参数说明 #  边缘 n 元分词过滤器可使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     min_gram 可选 整数 要生成的 n 元词项的最小长度。默认值为1。   max_gram 可选 整数 要生成的 n 元词项的最大长度。对于边缘 n 元分词过滤器，默认值为1；对于自定义分词过滤器，默认值为2。避免将此参数设置为较低的值。如果该值设置得过低，将只会生成非常短的 n 元词项，并且可能找不到搜索词。例如，将max_gram设置为 3，并且对单词“banana”建立索引，那么生成的最长词元将是“ban”。如果用户搜索“banana”，将不会返回任何匹配结果。你可以使用截断truncate分词过滤器作为搜索分析器来降低这种风险。   preserve_original 可选 布尔值 将原始词元包含在输出中。默认值为false。    参考样例 #  以下示例请求创建了一个名为 edge_ngram_example 的新索引，并配置了一个带有边缘 n 元过滤器的分词器："
---


# Edge N-gram 分词过滤器

`edge_ngram` 分词过滤器与 `ngram` 分词过滤器非常相似，二者都会将特定字符串拆分成不同长度的子字符串。不过，`edge_ngram` 分词过滤器仅从词元的开头（边缘）生成 n 元子字符串。在自动补全或前缀匹配等场景中，它特别有用，因为在这些场景里，你希望在用户输入单词或短语时就能匹配词项的开头部分。

## 相关指南（先读这些）

- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

边缘 n 元分词过滤器可使用以下参数进行配置。

| 参数                | 必需/可选 | 数据类型 | 描述                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `min_gram`          | 可选      | 整数     | 要生成的 n 元词项的最小长度。默认值为`1`。                                                                                                                                                                                                                                                                                                                                                                   |
| `max_gram`          | 可选      | 整数     | 要生成的 n 元词项的最大长度。对于边缘 n 元分词过滤器，默认值为`1`；对于自定义分词过滤器，默认值为`2`。避免将此参数设置为较低的值。如果该值设置得过低，将只会生成非常短的 n 元词项，并且可能找不到搜索词。例如，将`max_gram`设置为 3，并且对单词“banana”建立索引，那么生成的最长词元将是“ban”。如果用户搜索“banana”，将不会返回任何匹配结果。你可以使用截断`truncate`分词过滤器作为搜索分析器来降低这种风险。 |
| `preserve_original` | 可选      | 布尔值   | 将原始词元包含在输出中。默认值为`false`。                                                                                                                                                                                                                                                                                                                                                                    |

## 参考样例

以下示例请求创建了一个名为 `edge_ngram_example` 的新索引，并配置了一个带有边缘 n 元过滤器的分词器：

```
PUT /edge_ngram_example
{
  "settings": {
    "analysis": {
      "filter": {
        "my_edge_ngram": {
          "type": "edge_ngram",
          "min_gram": 3,
          "max_gram": 4
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "my_edge_ngram"]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /edge_ngram_example/_analyze
{
  "analyzer": "my_analyzer",
  "text": "slow green turtle"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "slo",
      "start_offset": 0,
      "end_offset": 4,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "slow",
      "start_offset": 0,
      "end_offset": 4,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "gre",
      "start_offset": 5,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "gree",
      "start_offset": 5,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "tur",
      "start_offset": 11,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "turt",
      "start_offset": 11,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

