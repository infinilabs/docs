---
title: "长度分词过滤器（Length）"
date: 0001-01-01
summary: "Length 分词过滤器 #  length 分词过滤器用于从词元流中移除那些不符合指定长度标准（最小和最大长度值）的词元。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参数说明 #  长度分词过滤器可以使用以下参数进行配置。
   参数 必填/可选 数据类型 描述     min 可选 整数 词元的最小长度。默认值为 0。   max 可选 整数 词元的最大长度。默认值为整数类型的最大值（2147483647）。    参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个带有长度过滤器的分词器：
PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;only_keep_4_to_10_characters&#34;: { &#34;tokenizer&#34;: &#34;whitespace&#34;, &#34;filter&#34;: [ &#34;length_4_to_10&#34; ] } }, &#34;filter&#34;: { &#34;length_4_to_10&#34;: { &#34;type&#34;: &#34;length&#34;, &#34;min&#34;: 4, &#34;max&#34;: 10 } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Length 分词过滤器

`length` 分词过滤器用于从词元流中移除那些不符合指定长度标准（最小和最大长度值）的词元。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

长度分词过滤器可以使用以下参数进行配置。

| 参数  | 必填/可选 | 数据类型 | 描述                                                       |
| ----- | --------- | -------- | ---------------------------------------------------------- |
| `min` | 可选      | 整数     | 词元的最小长度。默认值为 `0`。                             |
| `max` | 可选      | 整数     | 词元的最大长度。默认值为整数类型的最大值（`2147483647`）。 |

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个带有长度过滤器的分词器：

```
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "only_keep_4_to_10_characters": {
          "tokenizer": "whitespace",
          "filter": [ "length_4_to_10" ]
        }
      },
      "filter": {
        "length_4_to_10": {
          "type": "length",
          "min": 4,
          "max": 10
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
  "analyzer": "only_keep_4_to_10_characters",
  "text": "Easysearch is a great tool!"
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
      "type": "word",
      "position": 0
    },
    {
      "token": "great",
      "start_offset": 16,
      "end_offset": 21,
      "type": "word",
      "position": 3
    },
    {
      "token": "tool!",
      "start_offset": 22,
      "end_offset": 27,
      "type": "word",
      "position": 4
    }
  ]
}
```

