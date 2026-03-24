---
title: "条件分词过滤器（Condition）"
date: 0001-01-01
summary: "Condition 分词过滤器 #  condition 分词过滤器是一种特殊类型的过滤器，它允许你根据特定标准有条件地应用其他分词过滤器。这使得在文本分析过程中，你能更精准地控制何时应用特定的分词过滤器。你可以配置多个过滤器，并且只有当满足你定义的条件时，这些过滤器才会被应用。该分词过滤器在进行特定语言处理和特殊字符处理时非常有用。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参数说明 #  要使用条件分词过滤器，必须配置两个参数，具体如下：
   参数 必需/可选 数据类型 描述     filter 必需 数组 当满足由 script 参数定义的指定条件时，指定应对词元应用哪个分词过滤器。   script 必需 对象 配置一个内联脚本，该脚本定义了要应用 filter 参数中指定的过滤器所需满足的条件（仅接受内联脚本）。    参考样例 #  以下示例请求创建了一个名为 my_conditional_index 的新索引，并配置了一个带有条件过滤器的分词器。该过滤器会对任何包含字符序列 “um” 的词元应用小写（lowercase）字母过滤器。
PUT /my_conditional_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_conditional_filter&#34;: { &#34;type&#34;: &#34;condition&#34;, &#34;filter&#34;: [&#34;lowercase&#34;], &#34;script&#34;: { &#34;source&#34;: &#34;token."
---


# Condition 分词过滤器

`condition` 分词过滤器是一种特殊类型的过滤器，它允许你根据特定标准有条件地应用其他分词过滤器。这使得在文本分析过程中，你能更精准地控制何时应用特定的分词过滤器。你可以配置多个过滤器，并且只有当满足你定义的条件时，这些过滤器才会被应用。该分词过滤器在进行特定语言处理和特殊字符处理时非常有用。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

要使用条件分词过滤器，必须配置两个参数，具体如下：

| 参数     | 必需/可选 | 数据类型 | 描述                                                                                               |
| -------- | --------- | -------- | -------------------------------------------------------------------------------------------------- |
| `filter` | 必需      | 数组     | 当满足由 `script` 参数定义的指定条件时，指定应对词元应用哪个分词过滤器。                           |
| `script` | 必需      | 对象     | 配置一个内联脚本，该脚本定义了要应用 `filter` 参数中指定的过滤器所需满足的条件（仅接受内联脚本）。 |

## 参考样例

以下示例请求创建了一个名为 `my_conditional_index` 的新索引，并配置了一个带有条件过滤器的分词器。该过滤器会对任何包含字符序列 “um” 的词元应用小写（`lowercase`）字母过滤器。

```
PUT /my_conditional_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_conditional_filter": {
          "type": "condition",
          "filter": ["lowercase"],
          "script": {
            "source": "token.getTerm().toString().contains('um')"
          }
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "my_conditional_filter"
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
GET /my_conditional_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "THE BLACK CAT JUMPS OVER A LAZY DOG"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "THE",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "BLACK",
      "start_offset": 4,
      "end_offset": 9,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "CAT",
      "start_offset": 10,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "jumps",
      "start_offset": 14,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "OVER",
      "start_offset": 20,
      "end_offset": 24,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "A",
      "start_offset": 25,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "LAZY",
      "start_offset": 27,
      "end_offset": 31,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "DOG",
      "start_offset": 32,
      "end_offset": 35,
      "type": "<ALPHANUM>",
      "position": 7
    }
  ]
}
```

