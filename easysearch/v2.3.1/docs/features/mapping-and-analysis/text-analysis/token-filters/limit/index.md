---
title: "限制分词过滤器（Limit）"
date: 0001-01-01
summary: "Limit 分词过滤器 #  limit 分词过滤器用于限制分词链通过的词元数量。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参数说明 #  限制分词过滤器可以使用以下参数进行配置。
   参数 必填/可选 数据类型 描述     max_token_count 可选 整数 要生成的词元的最大数量。默认值为 1。   consume_all_tokens 可选 布尔值 （专家级设置）即使结果超过 max_token_count，也会使用来自词元生成器的所有词元。当设置此参数时，输出仍然只包含 max_token_count 指定数量的词元。不过，词元生成器生成的所有词元都会被处理。默认值为 false。    参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个带有限制过滤器的分析器：
PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;three_token_limit&#34;: { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;custom_token_limit&#34; ] } }, &#34;filter&#34;: { &#34;custom_token_limit&#34;: { &#34;type&#34;: &#34;limit&#34;, &#34;max_token_count&#34;: 3 } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Limit 分词过滤器

`limit` 分词过滤器用于限制分词链通过的词元数量。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

限制分词过滤器可以使用以下参数进行配置。

| 参数                 | 必填/可选 | 数据类型 | 描述                                                                                                                                                                                                          |
| -------------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `max_token_count`    | 可选      | 整数     | 要生成的词元的最大数量。默认值为 `1`。                                                                                                                                                                        |
| `consume_all_tokens` | 可选      | 布尔值   | （专家级设置）即使结果超过 `max_token_count`，也会使用来自词元生成器的所有词元。当设置此参数时，输出仍然只包含 `max_token_count` 指定数量的词元。不过，词元生成器生成的所有词元都会被处理。默认值为 `false`。 |

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个带有限制过滤器的分析器：

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "three_token_limit": {
          "tokenizer": "standard",
          "filter": [ "custom_token_limit" ]
        }
      },
      "filter": {
        "custom_token_limit": {
          "type": "limit",
          "max_token_count": 3
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
GET /my_index/_analyze
{
  "analyzer": "three_token_limit",
  "text": "Easysearch is a powerful and flexible search engine."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "Easysearch",
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
      "token": "a",
      "start_offset": 14,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

