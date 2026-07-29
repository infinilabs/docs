---
title: "保留词分词过滤器（Keep Words）"
date: 0001-01-01
summary: "Keep Words 分词过滤器 #  keep_words 分词过滤器旨在在分析过程中仅保留特定的词。如果你有大量文本，但只对某些关键字或术语感兴趣，那么这个过滤器就会很有用。
相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参数说明 #  词保留分词过滤器可以使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     keep_words 若未配置 keep_words_path 则为必需 字符串列表 要保留的词的列表。   keep_words_path 若未配置 keep_words 则为必需 字符串 包含要保留的词列表的文件的路径。   keep_words_case 可选 布尔值 在比较时是否将所有词转换为小写。默认值为 false。    参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个带有词保留过滤器的分词器：
PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;custom_keep_word&#34;: { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;keep_words_filter&#34; ] } }, &#34;filter&#34;: { &#34;keep_words_filter&#34;: { &#34;type&#34;: &#34;keep&#34;, &#34;keep_words&#34;: [&#34;example&#34;, &#34;world&#34;, &#34;easysearch&#34;], &#34;keep_words_case&#34;: true } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Keep Words 分词过滤器

`keep_words` 分词过滤器旨在在分析过程中仅保留特定的词。如果你有大量文本，但只对某些关键字或术语感兴趣，那么这个过滤器就会很有用。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

词保留分词过滤器可以使用以下参数进行配置。

| 参数              | 必需/可选                           | 数据类型   | 描述                                               |
| ----------------- | ----------------------------------- | ---------- | -------------------------------------------------- |
| `keep_words`      | 若未配置 `keep_words_path` 则为必需 | 字符串列表 | 要保留的词的列表。                                 |
| `keep_words_path` | 若未配置 `keep_words` 则为必需      | 字符串     | 包含要保留的词列表的文件的路径。                   |
| `keep_words_case` | 可选                                | 布尔值     | 在比较时是否将所有词转换为小写。默认值为 `false`。 |

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个带有词保留过滤器的分词器：

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_keep_word": {
          "tokenizer": "standard",
          "filter": [ "keep_words_filter" ]
        }
      },
      "filter": {
        "keep_words_filter": {
          "type": "keep",
          "keep_words": ["example", "world", "easysearch"],
          "keep_words_case": true
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
GET /my_index/_analyze
{
  "analyzer": "custom_keep_word",
  "text": "Hello, world! This is an Easysearch example."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "world",
      "start_offset": 7,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "Easysearch",
      "start_offset": 25,
      "end_offset": 35,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "example",
      "start_offset": 36,
      "end_offset": 43,
      "type": "<ALPHANUM>",
      "position": 6
    }
  ]
}
```

