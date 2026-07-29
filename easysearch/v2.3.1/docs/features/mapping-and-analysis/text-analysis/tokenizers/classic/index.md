---
title: "经典分词器（Classic）"
date: 0001-01-01
summary: "Classic 分词器 #  classic 分词器会解析文本，并应用英语语法规则将文本拆分为词元。它包含以下特定的逻辑来处理匹配规则：
相关指南（先读这些） #     文本分析：识别词元
   文本分析基础
  首字母缩写词
  电子邮件地址
  域名
  某些类型的标点符号
   这种词元生成器最适合处理英语文本。对于其他语言，尤其是那些具有不同语法结构的语言，它可能无法产生最佳效果。
 经典词元生成器按如下方式解析文本：
 标点符号：在大多数标点符号处拆分文本，并移除标点字符。后面不跟空格的点号会被视为词元的一部分。 连字符：在连字符处拆分单词，但当词元中存在数字时除外。当词元中存在数字时，该词元不会被拆分，而是被当作产品编号处理。 电子邮件：识别电子邮件地址和主机名，并将它们作为单个词元保留。  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个使用经典词元生成器的分词器：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_classic_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;classic&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_classic_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Classic 分词器

`classic` 分词器会解析文本，并应用英语语法规则将文本拆分为词元。它包含以下特定的逻辑来处理匹配规则：

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

- 首字母缩写词
- 电子邮件地址
- 域名
- 某些类型的标点符号

> 这种词元生成器最适合处理英语文本。对于其他语言，尤其是那些具有不同语法结构的语言，它可能无法产生最佳效果。

经典词元生成器按如下方式解析文本：

- **标点符号**：在大多数标点符号处拆分文本，并移除标点字符。后面不跟空格的点号会被视为词元的一部分。
- **连字符**：在连字符处拆分单词，但当词元中存在数字时除外。当词元中存在数字时，该词元不会被拆分，而是被当作产品编号处理。
- **电子邮件**：识别电子邮件地址和主机名，并将它们作为单个词元保留。

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个使用经典词元生成器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_classic_analyzer": {
          "type": "custom",
          "tokenizer": "classic"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_classic_analyzer"
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
  "analyzer": "my_classic_analyzer",
  "text": "For product AB3423, visit X&Y at example.com, email info@example.com, or call the operator's phone number 1-800-555-1234. P.S. 你好."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "For",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "product",
      "start_offset": 4,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "AB3423",
      "start_offset": 12,
      "end_offset": 18,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "visit",
      "start_offset": 20,
      "end_offset": 25,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "X&Y",
      "start_offset": 26,
      "end_offset": 29,
      "type": "<COMPANY>",
      "position": 4
    },
    {
      "token": "at",
      "start_offset": 30,
      "end_offset": 32,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "example.com",
      "start_offset": 33,
      "end_offset": 44,
      "type": "<HOST>",
      "position": 6
    },
    {
      "token": "email",
      "start_offset": 46,
      "end_offset": 51,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "info@example.com",
      "start_offset": 52,
      "end_offset": 68,
      "type": "<EMAIL>",
      "position": 8
    },
    {
      "token": "or",
      "start_offset": 70,
      "end_offset": 72,
      "type": "<ALPHANUM>",
      "position": 9
    },
    {
      "token": "call",
      "start_offset": 73,
      "end_offset": 77,
      "type": "<ALPHANUM>",
      "position": 10
    },
    {
      "token": "the",
      "start_offset": 78,
      "end_offset": 81,
      "type": "<ALPHANUM>",
      "position": 11
    },
    {
      "token": "operator's",
      "start_offset": 82,
      "end_offset": 92,
      "type": "<APOSTROPHE>",
      "position": 12
    },
    {
      "token": "phone",
      "start_offset": 93,
      "end_offset": 98,
      "type": "<ALPHANUM>",
      "position": 13
    },
    {
      "token": "number",
      "start_offset": 99,
      "end_offset": 105,
      "type": "<ALPHANUM>",
      "position": 14
    },
    {
      "token": "1-800-555-1234",
      "start_offset": 106,
      "end_offset": 120,
      "type": "<NUM>",
      "position": 15
    },
    {
      "token": "P.S.",
      "start_offset": 122,
      "end_offset": 126,
      "type": "<ACRONYM>",
      "position": 16
    },
    {
      "token": "你",
      "start_offset": 127,
      "end_offset": 128,
      "type": "<CJ>",
      "position": 17
    },
    {
      "token": "好",
      "start_offset": 128,
      "end_offset": 129,
      "type": "<CJ>",
      "position": 18
    }
  ]
}
```

## 词元类型

经典（`classic`）词元生成器产生的词元有以下类型：

### 词元类型及描述

| 词元类型        | 描述                                                                                                                               |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `<ALPHANUM>`    | 由字母、数字或两者组合而成的字母数字词元。                                                                                         |
| `<APOSTROPHE>`  | 包含撇号的词元，常用于所有格或缩写形式（例如 `John's`）。                                                                          |
| `<ACRONYM>`     | 首字母缩写词或缩写，通常以句点结尾（例如 `P.S.` 或 `U.S.A.`）。                                                                    |
| `<COMPANY>`     | 代表公司名称的词元（例如 `X&Y`）。如果这些词元没有自动生成，你可能需要进行自定义配置或使用过滤器。                                 |
| `<EMAIL>`       | 与电子邮件地址匹配的词元，包含 `@` 符号和域名（例如 `support@widgets.co` 或 `info@example.com`）。                                 |
| `<HOST>`        | 与网站或主机名匹配的词元，通常包含 `www.` 或诸如 `.com` 之类的域名后缀（例如 `www.example.com` 或 `example.org`）。                |
| `<NUM>`         | 仅包含数字或类似数字序列的词元（例如 `1-800`、`12345` 或 `3.14`）。                                                                |
| `<CJ>`          | 代表中文或日文字符的词元。                                                                                                         |
| `<ACRONYM_DEP>` | 已弃用的首字母缩写词处理方式（例如，旧版本中具有不同解析规则的首字母缩写词）。很少使用，主要用于与旧版词元生成器规则保持向后兼容。 |

