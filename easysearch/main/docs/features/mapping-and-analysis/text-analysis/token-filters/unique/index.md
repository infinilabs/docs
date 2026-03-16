---
title: "唯一分词过滤器（Unique）"
date: 0001-01-01
summary: "Unique 分词过滤器 #  unique 分词过滤器可确保在分词过程中仅保留唯一的词元，它会去除在单个字段或文本块中出现的重复词元。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参数说明 #  唯一分词过滤器可以使用以下参数进行配置：
   参数 必需/可选 数据类型 描述     only_on_same_position 可选 布尔值 如果设置为 true，该分词过滤器将充当去重分词过滤器，仅移除位于相同位置的词元。默认值为 false。    参考样例 #  以下示例请求创建了一个名为 unique_example 的新索引，并配置了一个带有唯一过滤器的分词器。
PUT /unique_example { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;unique_filter&#34;: { &#34;type&#34;: &#34;unique&#34;, &#34;only_on_same_position&#34;: false } }, &#34;analyzer&#34;: { &#34;unique_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;unique_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Unique 分词过滤器

`unique` 分词过滤器可确保在分词过程中仅保留唯一的词元，它会去除在单个字段或文本块中出现的重复词元。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

唯一分词过滤器可以使用以下参数进行配置：

| 参数                    | 必需/可选 | 数据类型 | 描述                                                                                              |
| ----------------------- | --------- | -------- | ------------------------------------------------------------------------------------------------- |
| `only_on_same_position` | 可选      | 布尔值   | 如果设置为 `true`，该分词过滤器将充当去重分词过滤器，仅移除位于相同位置的词元。默认值为 `false`。 |

## 参考样例

以下示例请求创建了一个名为 `unique_example` 的新索引，并配置了一个带有唯一过滤器的分词器。

```
PUT /unique_example
{
  "settings": {
    "analysis": {
      "filter": {
        "unique_filter": {
          "type": "unique",
          "only_on_same_position": false
        }
      },
      "analyzer": {
        "unique_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "unique_filter"
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
GET /unique_example/_analyze
{
  "analyzer": "unique_analyzer",
  "text": "Easysearch Easysearch is powerful powerful and scalable"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "easysearch",
      "start_offset": 0,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 22,
      "end_offset": 24,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "powerful",
      "start_offset": 25,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "and",
      "start_offset": 43,
      "end_offset": 46,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "scalable",
      "start_offset": 47,
      "end_offset": 55,
      "type": "<ALPHANUM>",
      "position": 4
    }
  ]
}
```

