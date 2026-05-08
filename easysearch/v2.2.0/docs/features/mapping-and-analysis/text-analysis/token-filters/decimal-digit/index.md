---
title: "十进制数字分词过滤器（Decimal Digit）"
date: 0001-01-01
summary: "Decimal Digit 分词过滤器 #  decimal_digit 分词过滤器用于将各种字符集中的十进制数字字符（0 到 9）规范化为它们对应的 ASCII 字符。当你希望在文本分析中确保所有数字都能被统一处理，而不管这些数字是以何种字符集书写时，这个过滤器就非常有用。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个带有十进制数字过滤器的分词器：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_decimal_digit_filter&#34;: { &#34;type&#34;: &#34;decimal_digit&#34; } }, &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;my_decimal_digit_filter&#34;] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /my_index/_analyze { &#34;analyzer&#34;: &#34;my_analyzer&#34;, &#34;text&#34;: &#34;123 ١٢٣ १२३&#34; } text分词："
---


# Decimal Digit 分词过滤器

`decimal_digit` 分词过滤器用于将各种字符集中的十进制数字字符（0 到 9）规范化为它们对应的 ASCII 字符。当你希望在文本分析中确保所有数字都能被统一处理，而不管这些数字是以何种字符集书写时，这个过滤器就非常有用。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个带有十进制数字过滤器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_decimal_digit_filter": {
          "type": "decimal_digit"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["my_decimal_digit_filter"]
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
  "analyzer": "my_analyzer",
  "text": "123 ١٢٣ १२३"
}
```

`text`分词：

- “123”（ASCII 数字）
- “١٢٣”（阿拉伯 - 印度数字）
- “१२३”（梵文数字）

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "123",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<NUM>",
      "position": 0
    },
    {
      "token": "123",
      "start_offset": 4,
      "end_offset": 7,
      "type": "<NUM>",
      "position": 1
    },
    {
      "token": "123",
      "start_offset": 8,
      "end_offset": 11,
      "type": "<NUM>",
      "position": 2
    }
  ]
}
```

