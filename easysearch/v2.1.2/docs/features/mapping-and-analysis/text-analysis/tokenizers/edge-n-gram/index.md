---
title: "边缘 N-gram 分词器（Edge N-gram）"
date: 0001-01-01
summary: "Edge N-gram 分词器 #  edge_ngram 分词器会从每个单词的开头生成部分单词词元，也就是 n 元组。它会根据指定的字符分割文本，并在定义的最小和最大长度范围内生成词元。这种分词器在实现即输即搜（search-as-you-type）功能时特别有用。
前缀 n 元组非常适合用于自动补全搜索，在这种搜索中单词的顺序可能会有所不同，例如在搜索产品名称或地址时。有关更多信息，请参阅&quot;自动补全&quot;相关内容。不过，对于像电影或歌曲标题这种顺序固定的文本，完成建议器（completion suggester）可能会更准确。
默认情况下，edge_ngram 分词器生成的词元最小长度为 1，最大长度为 2。例如，当分析文本 Easysearch 时，默认配置会生成 &ldquo;E&rdquo; 和 &ldquo;Ea&rdquo; 这两个 n 元组。这些短的 n 元组常常会匹配到太多不相关的词条，所以调整 n 元组的长度，对分词器进行优化是很有必要的。
相关指南（先读这些） #    文本分析：识别词元  部分匹配  文本分析基础  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个使用前缀 n 元词元生成器的分词器。该词元生成器生成长度为 3 到 6 个字符的词元，且将字母和符号都视为有效的词元字符。
PUT /edge_n_gram_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_custom_analyzer&#34;: { &#34;tokenizer&#34;: &#34;my_custom_tokenizer&#34; } }, &#34;tokenizer&#34;: { &#34;my_custom_tokenizer&#34;: { &#34;type&#34;: &#34;edge_ngram&#34;, &#34;min_gram&#34;: 3, &#34;max_gram&#34;: 6, &#34;token_chars&#34;: [ &#34;letter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Edge N-gram 分词器

`edge_ngram` 分词器会从每个单词的开头生成部分单词词元，也就是 n 元组。它会根据指定的字符分割文本，并在定义的最小和最大长度范围内生成词元。这种分词器在实现即输即搜（`search-as-you-type`）功能时特别有用。

前缀 n 元组非常适合用于自动补全搜索，在这种搜索中单词的顺序可能会有所不同，例如在搜索产品名称或地址时。有关更多信息，请参阅"自动补全"相关内容。不过，对于像电影或歌曲标题这种顺序固定的文本，完成建议器（`completion suggester`）可能会更准确。

默认情况下，`edge_ngram` 分词器生成的词元最小长度为 1，最大长度为 2。例如，当分析文本 `Easysearch` 时，默认配置会生成 "E" 和 "Ea" 这两个 n 元组。这些短的 n 元组常常会匹配到太多不相关的词条，所以调整 n 元组的长度，对分词器进行优化是很有必要的。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个使用前缀 n 元词元生成器的分词器。该词元生成器生成长度为 3 到 6 个字符的词元，且将字母和符号都视为有效的词元字符。

```
PUT /edge_n_gram_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "tokenizer": "my_custom_tokenizer"
        }
      },
      "tokenizer": {
        "my_custom_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 3,
          "max_gram": 6,
          "token_chars": [
            "letter"          ]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /edge_n_gram_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Code 42 rocks!"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "Cod",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 0
    },
    {
      "token": "Code",
      "start_offset": 0,
      "end_offset": 4,
      "type": "word",
      "position": 1
    },
    {
      "token": "roc",
      "start_offset": 8,
      "end_offset": 11,
      "type": "word",
      "position": 2
    },
    {
      "token": "rock",
      "start_offset": 8,
      "end_offset": 12,
      "type": "word",
      "position": 3
    },
    {
      "token": "rocks",
      "start_offset": 8,
      "end_offset": 13,
      "type": "word",
      "position": 4
    }
  ]
}
```

## 参数说明

| 参数                 | 必填/选填 | 数据类型   | 描述                                                                                                                                                                                                                                                                                                                                                                   |
| -------------------- | --------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `min_gram`           | 选填      | 整数       | 词元的最小长度。默认值为 1。                                                                                                                                                                                                                                                                                                                                           |
| `max_gram`           | 选填      | 整数       | 词元的最大长度。默认值为 2。                                                                                                                                                                                                                                                                                                                                           |
| `custom_token_chars` | 选填      | 字符串     | 被视为词元一部分的自定义字符（例如，`+-_`）。                                                                                                                                                                                                                                                                                                                          |
| `token_chars`        | 选填      | 字符串数组 | 定义要包含在词元中的字符类。词元会字符类规则进行拆分。默认是所有字符。可用的字符类包括：<br>- `letter`：字母字符（例如，a、ç 或 “京”）<br>- `digit`：数字字符（例如，3 或 7）<br>- `punctuation`：标点符号（例如，! 或 ?）<br>- `symbol`：其他符号（例如，$ 或 √）<br>- `whitespace`：空格或换行符<br>- `custom`：允许您在 `custom_token_chars` 设置中指定自定义字符。 |

## 最大词元长度（max_gram）参数的限制

最大词元长度（`max_gram`）参数设置了生成的词元的最大长度。当一个查询词项的长度超过这个设定时，它可能无法匹配索引中的任何词条。

例如，如果将 `max_gram` 设置为 4，那么在索引过程中，查询词 `explore` 会被分词为 `expl`。这样一来，当搜索完整的词条 `explore` 时，就无法匹配到已索引的词元 `expl` 了。

为了解决这个限制问题，你可以应用一个截断(`truncate`)词元过滤器，将搜索词缩短到最大词元长度。不过，这种方法也存在权衡取舍。将 `explore` 截断为 `expl` 可能会导致与一些不相关的词条匹配，比如 `explosion` 或 `explicit`，从而降低搜索精度。

我们建议仔细权衡 `max_gram` 的值，以确保在进行高效分词的同时，尽量减少不相关的匹配。如果搜索精度至关重要，可考虑其他策略，比如调整查询分词器或微调过滤器。

## 最佳实践

我们建议仅在索引创建阶段使用前缀 n 元词元生成器，以确保部分单词词元得以存储。在执行搜索时，应使用基础分词器来匹配所有的查询词条。

### 配置“即输即搜”功能

要实现即输即搜（`search-as-you-type`）功能，可在索引创建时使用前缀 n 元词元生成器，在搜索时使用一个执行最少处理操作的分词器。以下示例展示了这种实现方法。

使用前缀 n 元词元生成器创建一个索引：

```
PUT /my-autocomplete-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "autocomplete": {
          "tokenizer": "autocomplete",
          "filter": [
            "lowercase"
          ]
        },
        "autocomplete_search": {
          "tokenizer": "lowercase"
        }
      },
      "tokenizer": {
        "autocomplete": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10,
          "token_chars": [
            "letter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "autocomplete",
        "search_analyzer": "autocomplete_search"
      }
    }
  }
}
```

索引一个包含 `product` 字段的文档，并刷新该索引：

```
PUT my-autocomplete-index/_doc/1?refresh
{
  "title": "Laptop Pro"
}
```

这种配置确保了前缀 n 元词元生成器会将诸如 “Laptop” 这样的词条拆分成 “La”、“Lap” 和 “Lapt” 等词元，从而在搜索时能够实现部分匹配。在搜索阶段，`standard`词元生成器会简化查询操作，并且由于有小写字母过滤器的存在，还能确保匹配操作不区分大小写。

现在，当搜索 “laptop Pr” 或 “lap pr” 时，就会基于部分匹配检索到相关的文档：

```
GET my-autocomplete-index/_search
{
  "query": {
    "match": {
      "title": {
        "query": "lap pr",
        "operator": "and"
      }
    }
  }
}
```

了解更多信息，请参阅 ` Search as you type` 相关内容。

