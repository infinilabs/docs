---
title: "截断分词过滤器（Truncate）"
date: 0001-01-01
summary: "Truncate 分词过滤器 #  truncate 分词过滤器用于缩短超过指定长度的词元。它会将词元修剪至最大字符数，确保超过该限制的词元被截断。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参数说明 #  截断分词过滤器可以使用以下参数进行配置：
   参数 必需/可选 数据类型 描述     length 可选 整数 指定生成的词元的最大长度。默认值为 10。    参考样例 #  以下示例请求创建了一个名为 truncate_example 的新索引，并配置了一个带有截断过滤器的分词器。
PUT /truncate_example { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;truncate_filter&#34;: { &#34;type&#34;: &#34;truncate&#34;, &#34;length&#34;: 5 } }, &#34;analyzer&#34;: { &#34;truncate_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;truncate_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Truncate 分词过滤器

`truncate` 分词过滤器用于缩短超过指定长度的词元。它会将词元修剪至最大字符数，确保超过该限制的词元被截断。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

截断分词过滤器可以使用以下参数进行配置：

| 参数     | 必需/可选 | 数据类型 | 描述                                    |
| -------- | --------- | -------- | --------------------------------------- |
| `length` | 可选      | 整数     | 指定生成的词元的最大长度。默认值为 10。 |

## 参考样例

以下示例请求创建了一个名为 `truncate_example` 的新索引，并配置了一个带有截断过滤器的分词器。

```
PUT /truncate_example
{
  "settings": {
    "analysis": {
      "filter": {
        "truncate_filter": {
          "type": "truncate",
          "length": 5
        }
      },
      "analyzer": {
        "truncate_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "truncate_filter"
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
GET /truncate_example/_analyze
{
  "analyzer": "truncate_analyzer",
  "text": "Easysearch is powerful and scalable"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "easys",
      "start_offset": 0,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 11,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "power",
      "start_offset": 14,
      "end_offset": 22,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "and",
      "start_offset": 23,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "scala",
      "start_offset": 27,
      "end_offset": 35,
      "type": "<ALPHANUM>",
      "position": 4
    }
  ]
}
```

