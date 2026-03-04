---
title: "小写分词过滤器（Lowercase）"
date: 0001-01-01
summary: "Lowercase 分词过滤器 #  lowercase 分词过滤器用于将分词流中的所有字符转换为小写，从而使搜索不区分大小写。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参数 #  小写分词过滤器可以使用以下参数进行配置。
   参数 必填/可选 描述     language 可选 指定一个特定语言的分词过滤器。有效值为：
- 希腊语 greek
- 爱尔兰语irish
- 土耳其语turkish。
默认值是 Lucene 的小写过滤器。    参考样例 #  以下示例请求创建了一个名为 custom_lowercase_example 的新索引。它配置了一个带有小写过滤器的分析器，并指定语言为希腊语。
PUT /custom_lowercase_example { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;greek_lowercase_example&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;greek_lowercase&#34;] } }, &#34;filter&#34;: { &#34;greek_lowercase&#34;: { &#34;type&#34;: &#34;lowercase&#34;, &#34;language&#34;: &#34;greek&#34; } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Lowercase 分词过滤器

`lowercase` 分词过滤器用于将分词流中的所有字符转换为小写，从而使搜索不区分大小写。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

### 参数

小写分词过滤器可以使用以下参数进行配置。

| 参数       | 必填/可选 | 描述                                                                                                                                         |
| ---------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `language` | 可选      | 指定一个特定语言的分词过滤器。有效值为：<br>- 希腊语 `greek`<br>- 爱尔兰语`irish`<br>- 土耳其语`turkish`。<br>默认值是 Lucene 的小写过滤器。 |

## 参考样例

以下示例请求创建了一个名为 `custom_lowercase_example` 的新索引。它配置了一个带有小写过滤器的分析器，并指定语言为希腊语。

```
PUT /custom_lowercase_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "greek_lowercase_example": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["greek_lowercase"]
        }
      },
      "filter": {
        "greek_lowercase": {
          "type": "lowercase",
          "language": "greek"
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
GET /custom_lowercase_example/_analyze
{
  "analyzer": "greek_lowercase_example",
  "text": "Αθήνα ΕΛΛΑΔΑ"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "αθηνα",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "ελλαδα",
      "start_offset": 6,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

