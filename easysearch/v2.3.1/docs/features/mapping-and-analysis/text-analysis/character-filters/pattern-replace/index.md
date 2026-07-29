---
title: "正则替换字符过滤器（Pattern Replace）"
date: 0001-01-01
summary: "Pattern Replace 字符过滤器 #  pattern_replace 字符过滤器使你能够使用正则表达式来定义文本匹配替换的模式。对于文本转换的高阶需求场景，尤其是在处理复杂的字符串模式时，它是一种灵活的工具。
这个过滤器会用替换符合匹配模式的所有匹配项，从而可以轻松地对输入文本进行替换、删除或复杂的修改。你可以在分词之前使用它对输入内容进行规范化处理。
相关指南（先读这些） #    文本分析基础  文本分析：规范化  参考样例 #  为了规范电话号码，你可以使用正则表达式 [\\s()-]+去替换号码里的特殊格式：
 []：定义一个字符类，意味着它将匹配方括号内的任意一个字符。 \\s：匹配任何空白字符，如空格、制表符或换行符。 ()：匹配字面意义上的括号（( 或 )）。 -：匹配字面意义上的连字符（-）。 +：指定该模式应匹配前面字符的一次或多次出现。  模式 [\\s()-]+ 将匹配由一个或多个空白字符、括号或连字符组成的任意序列，并将其从输入文本中移除。这确保了电话号码得到规范处理，结果将仅包含数字。
以下请求通过移除空格、连字符和括号来规范电话号码：
GET /_analyze { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;char_filter&#34;: [ { &#34;type&#34;: &#34;pattern_replace&#34;, &#34;pattern&#34;: &#34;[\\s()-]+&#34;, &#34;replacement&#34;: &#34;&#34; } ], &#34;text&#34;: &#34;(555) 123-4567&#34; } 返回内容中包含生成的词元：
{ &#34;tokens&#34;: [ { &#34;token&#34;: &#34;5551234567&#34;, &#34;start_offset&#34;: 1, &#34;end_offset&#34;: 14, &#34;type&#34;: &#34;&lt;NUM&gt;&#34;, &#34;position&#34;: 0 } ] } 参数说明 #  pattern_replace 字符过滤器必须使用以下参数进行配置。"
---


# Pattern Replace 字符过滤器

`pattern_replace` 字符过滤器使你能够使用正则表达式来定义文本匹配替换的模式。对于文本转换的高阶需求场景，尤其是在处理复杂的字符串模式时，它是一种灵活的工具。

这个过滤器会用替换符合匹配模式的所有匹配项，从而可以轻松地对输入文本进行替换、删除或复杂的修改。你可以在分词之前使用它对输入内容进行规范化处理。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参考样例

为了规范电话号码，你可以使用正则表达式 `[\\s()-]+`去替换号码里的特殊格式：

- `[]`：定义一个字符类，意味着它将匹配方括号内的任意一个字符。
- `\\s`：匹配任何空白字符，如空格、制表符或换行符。
- `()`：匹配字面意义上的括号（`(` 或 `)`）。
- `-`：匹配字面意义上的连字符（`-`）。
- `+`：指定该模式应匹配前面字符的一次或多次出现。

模式 `[\\s()-]+` 将匹配由一个或多个空白字符、括号或连字符组成的任意序列，并将其从输入文本中移除。这确保了电话号码得到规范处理，结果将仅包含数字。

以下请求通过移除空格、连字符和括号来规范电话号码：

```
GET /_analyze
{
  "tokenizer": "standard",
  "char_filter": [
    {
      "type": "pattern_replace",
      "pattern": "[\\s()-]+",
      "replacement": ""
    }
  ],
  "text": "(555) 123-4567"
}
```

返回内容中包含生成的词元：

```
{
  "tokens": [
    {
      "token": "5551234567",
      "start_offset": 1,
      "end_offset": 14,
      "type": "<NUM>",
      "position": 0
    }
  ]
}
```

## 参数说明

`pattern_replace` 字符过滤器必须使用以下参数进行配置。

| 参数          | 必需/可选 | 数据类型 | 描述                                                                                         |
| ------------- | --------- | -------- | -------------------------------------------------------------------------------------------- |
| `pattern`     | 必需      | 字符串   | 用于匹配输入文本部分内容的正则表达式。过滤器会识别并匹配此模式以执行替换操作。               |
| `replacement` | 可选      | 字符串   | 用于替换匹配内容的字符串。使用空字符串（`""`）可移除匹配到的文本。默认值为空字符串（`""`）。 |

## 创建自定义分词器

以下请求创建一个索引，该索引带有一个配置了 `pattern_replace` 字符过滤器的自定义分词器。此过滤器会从数字中移除货币符号以及千位分隔符（包括欧洲的 “.” 和美国的 “,”）：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "pattern_char_filter"
          ]
        }
      },
      "char_filter": {
        "pattern_char_filter": {
          "type": "pattern_replace",
          "pattern": "[$€,.]",
          "replacement": ""
        }
      }
    }
  }
}
```

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Total: $ 1,200.50 and € 1.100,75"
}
```

返回内容中包含了生成的词元：

```
{
  "tokens": [
    {
      "token": "Total",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "120050",
      "start_offset": 9,
      "end_offset": 17,
      "type": "<NUM>",
      "position": 1
    },
    {
      "token": "and",
      "start_offset": 18,
      "end_offset": 21,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "110075",
      "start_offset": 24,
      "end_offset": 32,
      "type": "<NUM>",
      "position": 3
    }
  ]
}
```

### 使用捕获组

你可以在 `replacement` 参数中使用捕获组。例如，以下请求创建了一个自定义分词器，该分词器使用匹配替换字符过滤器将电话号码中的连字符替换为点号：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "pattern_char_filter"
          ]
        }
      },
      "char_filter": {
        "pattern_char_filter": {
          "type": "pattern_replace",
          "pattern": "(\\d+)-(?=\\d)",
          "replacement": "$1."
        }
      }
    }
  }
}
```

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Call me at 555-123-4567 or 555-987-6543"
}
```

返回内容中包含了生成的词元：

```
{
  "tokens": [
    {
      "token": "Call",
      "start_offset": 0,
      "end_offset": 4,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "me",
      "start_offset": 5,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "at",
      "start_offset": 8,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "555.123.4567",
      "start_offset": 11,
      "end_offset": 23,
      "type": "<NUM>",
      "position": 3
    },
    {
      "token": "or",
      "start_offset": 24,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "555.987.6543",
      "start_offset": 27,
      "end_offset": 39,
      "type": "<NUM>",
      "position": 5
    }
  ]
}
```

