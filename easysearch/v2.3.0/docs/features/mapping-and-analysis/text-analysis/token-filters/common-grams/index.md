---
title: "常见词组分词过滤器（Common Grams）"
date: 0001-01-01
summary: "Common Grams 分词过滤器 #  common_grams 分词过滤器通过保留文本中常见的短语（常用词组）来提高搜索的相关性。当处理某些语言或数据集时，特定的单词组合经常作为一个整体出现，如果将它们当作单独的词元处理，可能会影响搜索的相关性，此时该过滤器就非常有用。如果输入字符串中常用词，这个分词过滤器会同时生成它们的一元词组（单个词）和二元词组（两个词的组合）。
使用这个分词过滤器可以保持常用短语的完整性，从而提高搜索的相关性。这有助于更准确地匹配查询内容，尤其是对于频繁出现的单词组合。它还能通过减少不相关的匹配结果来提高搜索的精度。
相关指南（先读这些） #    文本分析：停用词  邻近匹配  文本分析：规范化   使用此过滤器时，你必须谨慎选择并维护常用词(common_words)列表。
 参数说明 #  常用词组分词过滤器可通过以下参数进行配置：
   参数 必需/可选 数据类型 描述     common_words 必需 字符串列表 一个应被视为常用词词组的单词列表。如果 common_words 参数是一个空列表，common_grams 分词过滤器将成为一个无操作过滤器，即它根本不会修改输入的词元。   ignore_case 可选 布尔值 指示过滤器在匹配常用词时是否应忽略大小写差异。默认值为 false。   query_mode 可选 布尔值 当设置为 true 时，应用以下规则：
- 从 common_words 生成的一元词组（单个词）不包含在输出中。
- 非常用词后跟常用词形成的二元词组会保留在输出中。
- 如果非常用词的一元词组后面紧接着一个常用词，则该一元词组会被排除。
- 如果非常用词出现在文本末尾且前面是一个常用词，其一元词组不包含在输出中。    参考样例 #  以下示例请求创建了一个名为 my_common_grams_index 的新索引，并配置了一个使用常用词组（common_grams）过滤器的分析器。"
---


# Common Grams 分词过滤器

`common_grams` 分词过滤器通过保留文本中常见的短语（常用词组）来提高搜索的相关性。当处理某些语言或数据集时，特定的单词组合经常作为一个整体出现，如果将它们当作单独的词元处理，可能会影响搜索的相关性，此时该过滤器就非常有用。如果输入字符串中常用词，这个分词过滤器会同时生成它们的一元词组（单个词）和二元词组（两个词的组合）。

使用这个分词过滤器可以保持常用短语的完整性，从而提高搜索的相关性。这有助于更准确地匹配查询内容，尤其是对于频繁出现的单词组合。它还能通过减少不相关的匹配结果来提高搜索的精度。

## 相关指南（先读这些）

- [文本分析：停用词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stopwords.md" >}})
- [邻近匹配]({{< relref "/docs/features/fulltext-search/proximity-matching.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

> 使用此过滤器时，你必须谨慎选择并维护常用词(`common_words`)列表。

## 参数说明

常用词组分词过滤器可通过以下参数进行配置：

| 参数           | 必需/可选 | 数据类型   | 描述                                                                                                                                                                                                                                                                                                               |
| -------------- | --------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `common_words` | 必需      | 字符串列表 | 一个应被视为常用词词组的单词列表。如果 `common_words` 参数是一个空列表，`common_grams` 分词过滤器将成为一个无操作过滤器，即它根本不会修改输入的词元。                                                                                                                                                              |
| `ignore_case`  | 可选      | 布尔值     | 指示过滤器在匹配常用词时是否应忽略大小写差异。默认值为 `false`。                                                                                                                                                                                                                                                   |
| `query_mode`   | 可选      | 布尔值     | 当设置为 `true` 时，应用以下规则：<br> - 从 `common_words` 生成的一元词组（单个词）不包含在输出中。<br> - 非常用词后跟常用词形成的二元词组会保留在输出中。<br> - 如果非常用词的一元词组后面紧接着一个常用词，则该一元词组会被排除。<br> - 如果非常用词出现在文本末尾且前面是一个常用词，其一元词组不包含在输出中。 |

## 参考样例

以下示例请求创建了一个名为 `my_common_grams_index` 的新索引，并配置了一个使用常用词组（`common_grams`）过滤器的分析器。

```
PUT /my_common_grams_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_common_grams_filter": {
          "type": "common_grams",
          "common_words": ["a", "in", "for"],
          "ignore_case": true,
          "query_mode": true
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_common_grams_filter"
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
GET /my_common_grams_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "A quick black cat jumps over the lazy dog in the park"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {"token": "a_quick","start_offset": 0,"end_offset": 7,"type": "gram","position": 0},
    {"token": "quick","start_offset": 2,"end_offset": 7,"type": "<ALPHANUM>","position": 1},
    {"token": "black","start_offset": 8,"end_offset": 13,"type": "<ALPHANUM>","position": 2},
    {"token": "cat","start_offset": 14,"end_offset": 17,"type": "<ALPHANUM>","position": 3},
    {"token": "jumps","start_offset": 18,"end_offset": 23,"type": "<ALPHANUM>","position": 4},
    {"token": "over","start_offset": 24,"end_offset": 28,"type": "<ALPHANUM>","position": 5},
    {"token": "the","start_offset": 29,"end_offset": 32,"type": "<ALPHANUM>","position": 6},
    {"token": "lazy","start_offset": 33,"end_offset": 37,"type": "<ALPHANUM>","position": 7},
    {"token": "dog_in","start_offset": 38,"end_offset": 44,"type": "gram","position": 8},
    {"token": "in_the","start_offset": 42,"end_offset": 48,"type": "gram","position": 9},
    {"token": "the","start_offset": 45,"end_offset": 48,"type": "<ALPHANUM>","position": 10},
    {"token": "park","start_offset": 49,"end_offset": 53,"type": "<ALPHANUM>","position": 11}
  ]
}
```

