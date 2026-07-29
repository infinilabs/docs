---
title: "关键字重复分词过滤器（Keyword Repeat）"
date: 0001-01-01
summary: "Keyword Repeat 分词过滤器 #  keyword_repeat 分词过滤器会将词元的关键词版本发送到词元流中。当你希望在经过进一步的词元转换（例如词干提取或同义词扩展）之后，既保留原始词元，又保留其修改后的版本时，通常会使用这个过滤器。这些重复的词元使得原始的、未改变的词元版本能够与修改后的版本一起保留在最终的分析结果中。
 注意：keyword_repeat 分词过滤器应该放置在词干提取过滤器之前。词干提取并非应用于每个词元，因此在词干提取之后，你可能会在同一位置得到重复的词元。为了去除重复词元，可以在词干提取器之后使用 remove_duplicates 分词过滤器。
 相关指南（先读这些） #    文本分析：词干提取  文本分析：规范化  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个带有关键词重复过滤器的分词器：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_kstem&#34;: { &#34;type&#34;: &#34;kstem&#34; }, &#34;my_lowercase&#34;: { &#34;type&#34;: &#34;lowercase&#34; } }, &#34;analyzer&#34;: { &#34;my_custom_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;my_lowercase&#34;, &#34;keyword_repeat&#34;, &#34;my_kstem&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Keyword Repeat 分词过滤器

`keyword_repeat` 分词过滤器会将词元的关键词版本发送到词元流中。当你希望在经过进一步的词元转换（例如词干提取或同义词扩展）之后，既保留原始词元，又保留其修改后的版本时，通常会使用这个过滤器。这些重复的词元使得原始的、未改变的词元版本能够与修改后的版本一起保留在最终的分析结果中。

> **注意**：`keyword_repeat` 分词过滤器应该放置在词干提取过滤器之前。词干提取并非应用于每个词元，因此在词干提取之后，你可能会在同一位置得到重复的词元。为了去除重复词元，可以在词干提取器之后使用 `remove_duplicates` 分词过滤器。

## 相关指南（先读这些）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个带有关键词重复过滤器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_kstem": {
          "type": "kstem"
        },
        "my_lowercase": {
          "type": "lowercase"
        }
      },
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "my_lowercase",
            "keyword_repeat",
            "my_kstem"
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
POST /my_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Stopped quickly"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "stopped",
      "start_offset": 0,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "stop",
      "start_offset": 0,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "quickly",
      "start_offset": 8,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "quick",
      "start_offset": 8,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

你可以通过在 `_analyze` 查询中添加以下参数，进一步研究关键词重复分词过滤器的影响：

```
POST /my_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Stopped quickly",
  "explain": true,
  "attributes": "keyword"
}
```

返回内容中包含了详细信息，例如分词情况、过滤操作以及特定分词过滤器的应用情况：

```
{
  "detail": {
    "custom_analyzer": true,
    "charfilters": [],
    "tokenizer": {
      "name": "standard",
      "tokens": [
        {
          "token": "Stopped",
          "start_offset": 0,
          "end_offset": 7,
          "type": "<ALPHANUM>",
          "position": 0
        },
        {
          "token": "quickly",
          "start_offset": 8,
          "end_offset": 15,
          "type": "<ALPHANUM>",
          "position": 1
        }
      ]
    },
    "tokenfilters": [
      {
        "name": "my_lowercase",
        "tokens": [
          {
            "token": "stopped",
            "start_offset": 0,
            "end_offset": 7,
            "type": "<ALPHANUM>",
            "position": 0
          },
          {
            "token": "quickly",
            "start_offset": 8,
            "end_offset": 15,
            "type": "<ALPHANUM>",
            "position": 1
          }
        ]
      },
      {
        "name": "keyword_repeat",
        "tokens": [
          {
            "token": "stopped",
            "start_offset": 0,
            "end_offset": 7,
            "type": "<ALPHANUM>",
            "position": 0,
            "keyword": true
          },
          {
            "token": "stopped",
            "start_offset": 0,
            "end_offset": 7,
            "type": "<ALPHANUM>",
            "position": 0,
            "keyword": false
          },
          {
            "token": "quickly",
            "start_offset": 8,
            "end_offset": 15,
            "type": "<ALPHANUM>",
            "position": 1,
            "keyword": true
          },
          {
            "token": "quickly",
            "start_offset": 8,
            "end_offset": 15,
            "type": "<ALPHANUM>",
            "position": 1,
            "keyword": false
          }
        ]
      },
      {
        "name": "my_kstem",
        "tokens": [
          {
            "token": "stopped",
            "start_offset": 0,
            "end_offset": 7,
            "type": "<ALPHANUM>",
            "position": 0,
            "keyword": true
          },
          {
            "token": "stop",
            "start_offset": 0,
            "end_offset": 7,
            "type": "<ALPHANUM>",
            "position": 0,
            "keyword": false
          },
          {
            "token": "quickly",
            "start_offset": 8,
            "end_offset": 15,
            "type": "<ALPHANUM>",
            "position": 1,
            "keyword": true
          },
          {
            "token": "quick",
            "start_offset": 8,
            "end_offset": 15,
            "type": "<ALPHANUM>",
            "position": 1,
            "keyword": false
          }
        ]
      }
    ]
  }
}
```

