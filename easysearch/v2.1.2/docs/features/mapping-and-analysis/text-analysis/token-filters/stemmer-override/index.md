---
title: "词干覆盖分词过滤器（Stemmer Override）"
date: 0001-01-01
summary: "Stemmer Override 分词过滤器 #  stemmer_override 分词过滤器允许你定义自定义的词干提取规则，这些规则可以覆盖像波特（Porter）或雪球（Snowball）等默认词干提取器的行为。当你希望对某些特定的单词应用特殊的词干提取方式，而这些单词可能无法被标准的词干提取算法正确处理时，这个过滤器就会非常有用。
相关指南（先读这些） #    文本分析：词干提取  文本分析：规范化  参数说明 #  词干覆盖分词过滤器必须使用以下参数中的一个进行配置。
   参数 数据类型 描述     rules 字符串 直接在设置中定义覆盖规则。   rules_path 字符串 指定包含自定义规则（映射）的文件的路径。该路径可以是绝对路径，也可以是相对于配置目录的相对路径。    参考样例 #  以下示例请求创建了一个名为 my-index 的新索引，并配置了一个带有词干覆盖过滤器的分词器。
PUT /my-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_stemmer_override_filter&#34;: { &#34;type&#34;: &#34;stemmer_override&#34;, &#34;rules&#34;: [ &#34;running, runner =&gt; run&#34;, &#34;bought =&gt; buy&#34;, &#34;best =&gt; good&#34; ] } }, &#34;analyzer&#34;: { &#34;my_custom_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;my_stemmer_override_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Stemmer Override 分词过滤器

`stemmer_override` 分词过滤器允许你定义自定义的词干提取规则，这些规则可以覆盖像波特（Porter）或雪球（Snowball）等默认词干提取器的行为。当你希望对某些特定的单词应用特殊的词干提取方式，而这些单词可能无法被标准的词干提取算法正确处理时，这个过滤器就会非常有用。

## 相关指南（先读这些）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

词干覆盖分词过滤器必须使用以下参数中的一个进行配置。

| 参数         | 数据类型 | 描述                                                                                             |
| ------------ | -------- | ------------------------------------------------------------------------------------------------ |
| `rules`      | 字符串   | 直接在设置中定义覆盖规则。                                                                       |
| `rules_path` | 字符串   | 指定包含自定义规则（映射）的文件的路径。该路径可以是绝对路径，也可以是相对于配置目录的相对路径。 |

## 参考样例

以下示例请求创建了一个名为 `my-index` 的新索引，并配置了一个带有词干覆盖过滤器的分词器。

```
PUT /my-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_stemmer_override_filter": {
          "type": "stemmer_override",
          "rules": [
            "running, runner => run",
            "bought => buy",
            "best => good"
          ]
        }
      },
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_stemmer_override_filter"
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
GET /my-index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "I am a runner and bought the best shoes"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "i",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "am",
      "start_offset": 2,
      "end_offset": 4,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "a",
      "start_offset": 5,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "run",
      "start_offset": 7,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "and",
      "start_offset": 14,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "buy",
      "start_offset": 18,
      "end_offset": 24,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "the",
      "start_offset": 25,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "good",
      "start_offset": 29,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "shoes",
      "start_offset": 34,
      "end_offset": 39,
      "type": "<ALPHANUM>",
      "position": 8
    }
  ]
}
```

