---
title: "谓词分词过滤器（Predicate Token Filter）"
date: 0001-01-01
summary: "Predicate Token Filter 分词过滤器 #  predicate_token_filter 分词过滤器会根据自定义脚本中定义的条件来评估词元是应该保留还是丢弃。词元的评估是在分析谓词上下文中进行的。此过滤器仅支持内联 Painless 脚本。
相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参数说明 #  谓词分词过滤器有一个必需参数：script。该参数提供一个条件，用于评估词元是否应被保留。
参考样例 #  以下示例请求创建了一个名为 predicate_index 的新索引，并配置了一个带有谓词分词过滤器的分词器。该过滤器指定仅输出长度超过 7 个字符的词元。
PUT /predicate_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_predicate_filter&#34;: { &#34;type&#34;: &#34;predicate_token_filter&#34;, &#34;script&#34;: { &#34;source&#34;: &#34;token.term.length() &gt; 7&#34; } } }, &#34;analyzer&#34;: { &#34;predicate_analyzer&#34;: { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;my_predicate_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Predicate Token Filter 分词过滤器

`predicate_token_filter` 分词过滤器会根据自定义脚本中定义的条件来评估词元是应该保留还是丢弃。词元的评估是在分析谓词上下文中进行的。此过滤器仅支持内联 Painless 脚本。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

谓词分词过滤器有一个必需参数：`script`。该参数提供一个条件，用于评估词元是否应被保留。

## 参考样例

以下示例请求创建了一个名为 `predicate_index` 的新索引，并配置了一个带有谓词分词过滤器的分词器。该过滤器指定仅输出长度超过 7 个字符的词元。

```
PUT /predicate_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_predicate_filter": {
          "type": "predicate_token_filter",
          "script": {
            "source": "token.term.length() > 7"
          }
        }
      },
      "analyzer": {
        "predicate_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_predicate_filter"
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
POST /predicate_index/_analyze
{
  "text": "The Easysearch community is growing rapidly",
  "analyzer": "predicate_analyzer"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "easysearch",
      "start_offset": 4,
      "end_offset": 14,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "community",
      "start_offset": 15,
      "end_offset": 24,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

