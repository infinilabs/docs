---
title: "停用词分词过滤器（Stop）"
date: 0001-01-01
summary: "Stop 分词过滤器 #  stop 分词过滤器用于在分词过程中从词元流中去除常见词汇（也称为停用词）。停用词通常是冠词和介词，比如 &ldquo;a&rdquo; 或 &ldquo;for&rdquo;。这些词在搜索查询中意义不大，并且常常被排除在外，以提高搜索效率和相关性。
默认的英语停用词列表包含以下单词：a、an、and、are、as、at、be、but、by、for、if、in、into、is、it、no、not、of、on、or、such、that、the、their、then、there、these、they、this、to、was、will 以及 with。
相关指南（先读这些） #    停用词  文本分析基础  参数说明 #  停用词分词过滤器可以使用以下参数进行配置：
   参数 必需/可选 数据类型 描述     stopwords 可选 字符串 既可以指定自定义的停用词数组，也可以指定一种语言以获取预定义的 Lucene 停用词列表。可指定的语言如下：
- _arabic_
- _armenian_
- _basque_
- _bengali_
- _brazilian_（巴西葡萄牙语）
- _bulgarian_
- _catalan_
- _cjk_（中文、日语和韩语）
- _czech_
- _danish_
- _dutch_
- _english_（默认值）
- _estonian_
- _finnish_"
---


# Stop 分词过滤器

`stop` 分词过滤器用于在分词过程中从词元流中去除常见词汇（也称为停用词）。停用词通常是冠词和介词，比如 "a" 或 "for"。这些词在搜索查询中意义不大，并且常常被排除在外，以提高搜索效率和相关性。

默认的英语停用词列表包含以下单词：a、an、and、are、as、at、be、but、by、for、if、in、into、is、it、no、not、of、on、or、such、that、the、their、then、there、these、they、this、to、was、will 以及 with。

## 相关指南（先读这些）

- [停用词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stopwords.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参数说明

停用词分词过滤器可以使用以下参数进行配置：

| 参数             | 必需/可选 | 数据类型 | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `stopwords`      | 可选      | 字符串   | 既可以指定自定义的停用词数组，也可以指定一种语言以获取预定义的 Lucene 停用词列表。可指定的语言如下：<br> - `_arabic_`<br> - `_armenian_`<br> - `_basque_`<br> - `_bengali_`<br> - `_brazilian_`（巴西葡萄牙语）<br> - `_bulgarian_`<br> - `_catalan_`<br> - `_cjk_`（中文、日语和韩语）<br> - `_czech_`<br> - `_danish_`<br> - `_dutch_`<br> - `_english_`（默认值）<br> - `_estonian_`<br> - `_finnish_`<br> - `_french_`<br> - `_galician_`<br> - `_german_`<br> - `_greek_`<br> - `_hindi_`<br> - `_hungarian_`<br> - `_indonesian_`<br> - `_irish_`<br> - `_italian_`<br> - `_latvian_`<br> - `_lithuanian_`<br> - `_norwegian_`<br> - `_persian_`<br> - `_portuguese_`<br> - `_romanian_`<br> - `_russian_`<br> - `_sorani_`<br> - `_spanish_`<br> - `_swedish_`<br> - `_thai_`<br> - `_turkish_` |
| `stopwords_path` | 可选      | 字符串   | 指定包含自定义停用词的文件的路径（可以是绝对路径，也可以是相对于配置目录的相对路径）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `ignore_case`    | 可选      | 布尔值   | 如果设置为 `true`，则在匹配停用词时不区分大小写。默认值为 `false`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| remove_trailing  | 可选      | 布尔值   | 如果设置为 `true`，则在分析过程中会移除末尾的停用词。默认值为 `true`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

## 参考样例

以下示例请求创建了一个名为 `my-stopword-index` 的新索引，并配置了一个带有停用词过滤器的分词器，该过滤器使用预定义的英语停用词列表。

```
PUT /my-stopword-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_stop_filter": {
          "type": "stop",
          "stopwords": "_english_"
        }
      },
      "analyzer": {
        "my_stop_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_stop_filter"
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
GET /my-stopword-index/_analyze
{
  "analyzer": "my_stop_analyzer",
  "text": "A quick dog jumps over the turtle"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "quick",
      "start_offset": 2,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "dog",
      "start_offset": 8,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "jumps",
      "start_offset": 12,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "over",
      "start_offset": 18,
      "end_offset": 22,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "turtle",
      "start_offset": 27,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 6
    }
  ]
}
```

