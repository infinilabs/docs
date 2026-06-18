---
title: "去重分词过滤器（Remove Duplicates）"
date: 0001-01-01
summary: "Remove Duplicates 分词过滤器 #  remove_duplicates 分词过滤器用于去除在分词过程中在相同位置生成的重复词元。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参考样例 #  以下示例请求创建了一个带有 keyword_repeat（关键词重复）分词过滤器的索引。该过滤器会在每个词元自身所在的相同位置添加该词元的关键词版本，然后使用 kstem（K 词干提取）过滤器来创建该词元的词干形式。
PUT /example-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;custom_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;keyword_repeat&#34;, &#34;kstem&#34; ] } } } } } 使用以下请求来分析字符串 “Slower turtle”。
GET /example-index/_analyze { &#34;analyzer&#34;: &#34;custom_analyzer&#34;, &#34;text&#34;: &#34;Slower turtle&#34; } 返回内容中在同一位置包含了两次词元 “turtle”。
{ &#34;tokens&#34;: [ { &#34;token&#34;: &#34;slower&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 6, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 0 }, { &#34;token&#34;: &#34;slow&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 6, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 0 }, { &#34;token&#34;: &#34;turtle&#34;, &#34;start_offset&#34;: 7, &#34;end_offset&#34;: 13, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 1 }, { &#34;token&#34;: &#34;turtle&#34;, &#34;start_offset&#34;: 7, &#34;end_offset&#34;: 13, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 1 } ] } 可以通过在索引设置中添加一个去重分词过滤器来移除重复的词元。"
---


# Remove Duplicates 分词过滤器

`remove_duplicates` 分词过滤器用于去除在分词过程中在相同位置生成的重复词元。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下示例请求创建了一个带有 `keyword_repeat`（关键词重复）分词过滤器的索引。该过滤器会在每个词元自身所在的相同位置添加该词元的关键词版本，然后使用 `kstem`（K 词干提取）过滤器来创建该词元的词干形式。

```
PUT /example-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "keyword_repeat",
            "kstem"
          ]
        }
      }
    }
  }
}
```

使用以下请求来分析字符串 “Slower turtle”。

```
GET /example-index/_analyze
{
  "analyzer": "custom_analyzer",
  "text": "Slower turtle"
}
```

返回内容中在同一位置包含了两次词元 “turtle”。

```
{
  "tokens": [
    {
      "token": "slower",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "slow",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "turtle",
      "start_offset": 7,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "turtle",
      "start_offset": 7,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

可以通过在索引设置中添加一个去重分词过滤器来移除重复的词元。

```
PUT /index-remove-duplicate
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "keyword_repeat",
            "kstem",
            "remove_duplicates"
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
GET /index-remove-duplicate/_analyze
{
  "analyzer": "custom_analyzer",
  "text": "Slower turtle"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "slower",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "slow",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "turtle",
      "start_offset": 7,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

