---
title: "URL 邮箱分词器（UAX URL Email）"
date: 0001-01-01
summary: "UAX URL Email 分词器 #  除了常规文本之外，uax_url_email 分词器还专门用于处理网址、电子邮件地址和域名。它基于 Unicode 文本分割算法（ UAX #29），这使得它能够正确地对复杂文本（包括网址和电子邮件地址）进行分词处理。
相关指南（先读这些） #    文本分析：识别词元  文本分析基础  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个使用 UAX URL 邮件词元生成器的分词器：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;uax_url_email_tokenizer&#34;: { &#34;type&#34;: &#34;uax_url_email&#34; } }, &#34;analyzer&#34;: { &#34;my_uax_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;uax_url_email_tokenizer&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_uax_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# UAX URL Email 分词器

除了常规文本之外，`uax_url_email` 分词器还专门用于处理网址、电子邮件地址和域名。它基于 Unicode 文本分割算法（[UAX #29](https://www.unicode.org/reports/tr29/)），这使得它能够正确地对复杂文本（包括网址和电子邮件地址）进行分词处理。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个使用 UAX URL 邮件词元生成器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "uax_url_email_tokenizer": {
          "type": "uax_url_email"
        }
      },
      "analyzer": {
        "my_uax_analyzer": {
          "type": "custom",
          "tokenizer": "uax_url_email_tokenizer"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_uax_analyzer"
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
  "analyzer": "my_uax_analyzer",
  "text": "Contact us at support@example.com or visit https://example.com for details."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {"token": "Contact","start_offset": 0,"end_offset": 7,"type": "<ALPHANUM>","position": 0},
    {"token": "us","start_offset": 8,"end_offset": 10,"type": "<ALPHANUM>","position": 1},
    {"token": "at","start_offset": 11,"end_offset": 13,"type": "<ALPHANUM>","position": 2},
    {"token": "support@example.com","start_offset": 14,"end_offset": 33,"type": "<EMAIL>","position": 3},
    {"token": "or","start_offset": 34,"end_offset": 36,"type": "<ALPHANUM>","position": 4},
    {"token": "visit","start_offset": 37,"end_offset": 42,"type": "<ALPHANUM>","position": 5},
    {"token": "https://example.com","start_offset": 43,"end_offset": 62,"type": "<URL>","position": 6},
    {"token": "for","start_offset": 63,"end_offset": 66,"type": "<ALPHANUM>","position": 7},
    {"token": "details","start_offset": 67,"end_offset": 74,"type": "<ALPHANUM>","position": 8}
  ]
}
```

## 参数说明

UAX URL 邮件词元生成器可以使用以下参数进行配置。

| 参数               | 必需/可选 | 数据类型 | 描述                                                                                                           |
| ------------------ | --------- | -------- | -------------------------------------------------------------------------------------------------------------- |
| `max_token_length` | 可选      | 整数     | 设置生成词元的最大长度。若超过该长度，词元将按照 `max_token_length` 所配置的长度拆分为多个词元。默认值为 255。 |

