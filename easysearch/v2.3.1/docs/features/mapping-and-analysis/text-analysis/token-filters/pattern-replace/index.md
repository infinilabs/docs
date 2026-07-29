---
title: "正则替换分词过滤器（Pattern Replace）"
date: 0001-01-01
summary: "Pattern Replace 分词过滤器 #  pattern_replace 分词过滤器允许您使用正则表达式来修改词元。此过滤器会将词元中的模式替换为指定的值，这使您在对词元进行索引之前，能够灵活地转换或规范化词元。当您在分析过程中需要清理或规范文本时，该过滤器尤其有用。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参数说明 #  匹配替换分词过滤器可以使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     pattern 必需 字符串 一个正则表达式模式，用于匹配需要被替换的文本。   all 可选 布尔值 是否替换所有匹配的模式。如果为 false，则仅替换第一个匹配项。默认值为 true。   replacement 可选 字符串 用于替换匹配模式的字符串。默认值为空字符串。    参考样例 #  以下示例请求创建了一个名为 text_index 的新索引，并配置了一个带有匹配替换过滤器的分词器，用于将包含数字的词元替换为字符串 [NUM]：
PUT /text_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;number_replace_filter&#34;: { &#34;type&#34;: &#34;pattern_replace&#34;, &#34;pattern&#34;: &#34;\\d+&#34;, &#34;replacement&#34;: &#34;[NUM]&#34; } }, &#34;analyzer&#34;: { &#34;number_analyzer&#34;: { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;number_replace_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Pattern Replace 分词过滤器

`pattern_replace` 分词过滤器允许您使用正则表达式来修改词元。此过滤器会将词元中的模式替换为指定的值，这使您在对词元进行索引之前，能够灵活地转换或规范化词元。当您在分析过程中需要清理或规范文本时，该过滤器尤其有用。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

匹配替换分词过滤器可以使用以下参数进行配置。

| 参数          | 必需/可选 | 数据类型 | 描述                                                                        |
| ------------- | --------- | -------- | --------------------------------------------------------------------------- |
| `pattern`     | 必需      | 字符串   | 一个正则表达式模式，用于匹配需要被替换的文本。                              |
| `all`         | 可选      | 布尔值   | 是否替换所有匹配的模式。如果为 false，则仅替换第一个匹配项。默认值为 true。 |
| `replacement` | 可选      | 字符串   | 用于替换匹配模式的字符串。默认值为空字符串。                                |

## 参考样例

以下示例请求创建了一个名为 `text_index` 的新索引，并配置了一个带有匹配替换过滤器的分词器，用于将包含数字的词元替换为字符串 `[NUM]`：

```
PUT /text_index
{
  "settings": {
    "analysis": {
      "filter": {
        "number_replace_filter": {
          "type": "pattern_replace",
          "pattern": "\\d+",
          "replacement": "[NUM]"
        }
      },
      "analyzer": {
        "number_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "number_replace_filter"
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
POST /text_index/_analyze
{
  "text": "Visit us at 98765 Example St.",
  "analyzer": "number_analyzer"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "visit",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "us",
      "start_offset": 6,
      "end_offset": 8,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "at",
      "start_offset": 9,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "[NUM]",
      "start_offset": 12,
      "end_offset": 17,
      "type": "<NUM>",
      "position": 3
    },
    {
      "token": "example",
      "start_offset": 18,
      "end_offset": 25,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "st",
      "start_offset": 26,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 5
    }
  ]
}
```

