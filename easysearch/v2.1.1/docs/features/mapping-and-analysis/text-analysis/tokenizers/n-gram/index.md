---
title: "N-gram 分词器（N-gram）"
date: 0001-01-01
summary: "N-gram 分词器 #  ngram 分词器会将文本拆分为指定长度的重叠 n-gram（固定长度为 n 的字符序列）。当你希望实现部分单词匹配或自动补全搜索功能时，这个分词器特别有用，因为它会生成原始输入文本的子字符串（n-gram 字符串）。
相关指南（先读这些） #    文本分析：识别词元  部分匹配  文本分析基础  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个使用 n-gram 词元生成器的分词器。
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_ngram_tokenizer&#34;: { &#34;type&#34;: &#34;ngram&#34;, &#34;min_gram&#34;: 3, &#34;max_gram&#34;: 4, &#34;token_chars&#34;: [&#34;letter&#34;, &#34;digit&#34;] } }, &#34;analyzer&#34;: { &#34;my_ngram_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;my_ngram_tokenizer&#34; } } } } } 生成的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /my_index/_analyze { &#34;analyzer&#34;: &#34;my_ngram_analyzer&#34;, &#34;text&#34;: &#34;Search&#34; } 返回内容包含产生的词元"
---


# N-gram 分词器

`ngram` 分词器会将文本拆分为指定长度的重叠 n-gram（固定长度为 n 的字符序列）。当你希望实现部分单词匹配或自动补全搜索功能时，这个分词器特别有用，因为它会生成原始输入文本的子字符串（n-gram 字符串）。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个使用 n-gram 词元生成器的分词器。

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_ngram_tokenizer": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 4,
          "token_chars": ["letter", "digit"]
        }
      },
      "analyzer": {
        "my_ngram_analyzer": {
          "type": "custom",
          "tokenizer": "my_ngram_tokenizer"
        }
      }
    }
  }
}
```

## 生成的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index/_analyze
{
  "analyzer": "my_ngram_analyzer",
  "text": "Search"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {"token": "Sea","start_offset": 0,"end_offset": 3,"type": "word","position": 0},
    {"token": "Sear","start_offset": 0,"end_offset": 4,"type": "word","position": 1},
    {"token": "ear","start_offset": 1,"end_offset": 4,"type": "word","position": 2},
    {"token": "earc","start_offset": 1,"end_offset": 5,"type": "word","position": 3},
    {"token": "arc","start_offset": 2,"end_offset": 5,"type": "word","position": 4},
    {"token": "arch","start_offset": 2,"end_offset": 6,"type": "word","position": 5},
    {"token": "rch","start_offset": 3,"end_offset": 6,"type": "word","position": 6}
  ]
}
```

## 参数说明

N-gram 词元生成器可以使用以下参数进行配置。

| 参数                 | 必需/可选 | 数据类型   | 描述                                                                                                                                                                                                                                                                           |
| -------------------- | --------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `min_gram`           | 可选      | 整数       | n-gram 的最小长度。默认值为 1。                                                                                                                                                                                                                                                |
| `max_gram`           | 可选      | 整数       | n-gram 的最大长度。默认值为 2。                                                                                                                                                                                                                                                |
| `token_chars`        | 可选      | 字符串列表 | 分词时要包含的字符类。有效值为：<br> - `letter`（字母）<br> - `digit`（数字）<br> - `whitespace`（空白字符）<br> - `punctuation`（标点符号）<br> - `symbol`（符号）<br> - `custom`（自定义，你还必须指定 `custom_token_chars` 参数）<br> 默认值为空列表 `[]`，即保留所有字符。 |
| `custom_token_chars` | 可选      | 字符串     | 要包含在词元中的自定义字符。                                                                                                                                                                                                                                                   |

### 关于 min_gram 与 max_gram 的最大差值

`min_gram` 和 `max_gram` 之间的最大差值可通过索引级别的 `index.max_ngram_diff` 设置进行配置，其默认值为 1。

以下示例请求创建了一个带有自定义 `index.max_ngram_diff` 设置的索引：

```
PUT /my-index
{
  "settings": {
    "index.max_ngram_diff": 2,
    "analysis": {
      "tokenizer": {
        "my_ngram_tokenizer": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 5,
          "token_chars": ["letter", "digit"]
        }
      },
      "analyzer": {
        "my_ngram_analyzer": {
          "type": "custom",
          "tokenizer": "my_ngram_tokenizer"
        }
      }
    }
  }
}
```

