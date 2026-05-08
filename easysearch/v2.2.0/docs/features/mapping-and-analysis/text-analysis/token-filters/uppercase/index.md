---
title: "大写分词过滤器（Uppercase）"
date: 0001-01-01
summary: "Uppercase 分词过滤器 #  uppercase 分词过滤器用于在分析过程中将所有词元（单词）转换为大写形式。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参考样例 #  以下示例请求创建了一个名为 uppercase_example 的新索引，并配置了一个带有大写过滤器的分词器。
PUT /uppercase_example { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;uppercase_filter&#34;: { &#34;type&#34;: &#34;uppercase&#34; } }, &#34;analyzer&#34;: { &#34;uppercase_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;uppercase_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元：
GET /uppercase_example/_analyze { &#34;analyzer&#34;: &#34;uppercase_analyzer&#34;, &#34;text&#34;: &#34;Easysearch is powerful&#34; } 返回内容包含产生的词元
{ &#34;tokens&#34;: [ { &#34;token&#34;: &#34;EASYSEARCH&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 10, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 0 }, { &#34;token&#34;: &#34;IS&#34;, &#34;start_offset&#34;: 11, &#34;end_offset&#34;: 13, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 1 }, { &#34;token&#34;: &#34;POWERFUL&#34;, &#34;start_offset&#34;: 14, &#34;end_offset&#34;: 22, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 2 } ] } "
---


# Uppercase 分词过滤器

`uppercase` 分词过滤器用于在分析过程中将所有词元（单词）转换为大写形式。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下示例请求创建了一个名为 `uppercase_example` 的新索引，并配置了一个带有大写过滤器的分词器。

```
PUT /uppercase_example
{
  "settings": {
    "analysis": {
      "filter": {
        "uppercase_filter": {
          "type": "uppercase"
        }
      },
      "analyzer": {
        "uppercase_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "uppercase_filter"
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
GET /uppercase_example/_analyze
{
  "analyzer": "uppercase_analyzer",
  "text": "Easysearch is powerful"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "EASYSEARCH",
      "start_offset": 0,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "IS",
      "start_offset": 11,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "POWERFUL",
      "start_offset": 14,
      "end_offset": 22,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

