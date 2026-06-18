---
title: "正则捕获分词过滤器（Pattern Capture）"
date: 0001-01-01
summary: "Pattern Capture 分词过滤器 #  pattern_capture 分词过滤器是一种功能强大的过滤器，它使用正则表达式根据特定模式来捕获和提取文本的部分内容。当你想要提取词元的特定部分，例如电子邮件域名、话题标签或数字，并将其重新用于进一步的分析或索引编制时，这个过滤器会非常有用。
相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参数说明 #  捕获匹配分词过滤器可以使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     patterns 必需 字符串数组 用于捕获文本部分的正则表达式数组。   preserve_original 必需 布尔值 是否在输出中保留原始词元。默认值为 true。    参考样例 #  以下示例请求创建了一个名为 email_index 的新索引，并配置了一个带有捕获匹配过滤器的分词器，以便从电子邮件地址中提取本地部分和域名。
PUT /email_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;email_pattern_capture&#34;: { &#34;type&#34;: &#34;pattern_capture&#34;, &#34;preserve_original&#34;: true, &#34;patterns&#34;: [ &#34;^([^@]+)&#34;, &#34;@(.+)$&#34; ] } }, &#34;analyzer&#34;: { &#34;email_analyzer&#34;: { &#34;tokenizer&#34;: &#34;uax_url_email&#34;, &#34;filter&#34;: [ &#34;email_pattern_capture&#34;, &#34;lowercase&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Pattern Capture 分词过滤器

`pattern_capture` 分词过滤器是一种功能强大的过滤器，它使用正则表达式根据特定模式来捕获和提取文本的部分内容。当你想要提取词元的特定部分，例如电子邮件域名、话题标签或数字，并将其重新用于进一步的分析或索引编制时，这个过滤器会非常有用。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

捕获匹配分词过滤器可以使用以下参数进行配置。

| 参数                | 必需/可选 | 数据类型   | 描述                                      |
| ------------------- | --------- | ---------- | ----------------------------------------- |
| `patterns`          | 必需      | 字符串数组 | 用于捕获文本部分的正则表达式数组。        |
| `preserve_original` | 必需      | 布尔值     | 是否在输出中保留原始词元。默认值为 true。 |

## 参考样例

以下示例请求创建了一个名为 `email_index` 的新索引，并配置了一个带有捕获匹配过滤器的分词器，以便从电子邮件地址中提取本地部分和域名。

```
PUT /email_index
{
  "settings": {
    "analysis": {
      "filter": {
        "email_pattern_capture": {
          "type": "pattern_capture",
          "preserve_original": true,
          "patterns": [
            "^([^@]+)",
            "@(.+)$"
          ]
        }
      },
      "analyzer": {
        "email_analyzer": {
          "tokenizer": "uax_url_email",
          "filter": [
            "email_pattern_capture",
            "lowercase"
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
POST /email_index/_analyze
{
  "text": "john.doe@example.com",
  "analyzer": "email_analyzer"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "john.doe@example.com",
      "start_offset": 0,
      "end_offset": 20,
      "type": "<EMAIL>",
      "position": 0
    },
    {
      "token": "john.doe",
      "start_offset": 0,
      "end_offset": 20,
      "type": "<EMAIL>",
      "position": 0
    },
    {
      "token": "example.com",
      "start_offset": 0,
      "end_offset": 20,
      "type": "<EMAIL>",
      "position": 0
    }
  ]
}
```

