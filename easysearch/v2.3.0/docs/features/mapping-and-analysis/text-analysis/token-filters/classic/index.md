---
title: "经典分词过滤器（Classic）"
date: 0001-01-01
summary: "Classic 分词过滤器 #  classic 分词过滤器的主要功能是与 classic 分词器配合使用。它通过应用以下常见的转换操作来处理词元，这些操作有助于文本分析和搜索：
 移除所有格词尾，例如 &ldquo;&rsquo;s&rdquo;。比如，&ldquo;John&rsquo;s&rdquo; 会变为 &ldquo;John&rdquo;。 从首字母缩略词中移除句点。例如，&ldquo;D.A.R.P.A.&rdquo; 会变为 &ldquo;DARPA&rdquo;。  相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参考样例 #  以下示例请求创建了一个名为 custom_classic_filter 的新索引，并配置了一个使用经典分词过滤器的分词器。
PUT /custom_classic_filter { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;custom_classic&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;classic&#34;, &#34;filter&#34;: [&#34;classic&#34;] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /custom_classic_filter/_analyze { &#34;analyzer&#34;: &#34;custom_classic&#34;, &#34;text&#34;: &#34;John&#39;s co-operate was excellent.&#34; } 返回内容包含产生的词元"
---


# Classic 分词过滤器

`classic` 分词过滤器的主要功能是与 `classic` 分词器配合使用。它通过应用以下常见的转换操作来处理词元，这些操作有助于文本分析和搜索：

- 移除所有格词尾，例如 "'s"。比如，"John's" 会变为 "John"。
- 从首字母缩略词中移除句点。例如，"D.A.R.P.A." 会变为 "DARPA"。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下示例请求创建了一个名为 `custom_classic_filter` 的新索引，并配置了一个使用经典分词过滤器的分词器。

```
PUT /custom_classic_filter
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_classic": {
          "type": "custom",
          "tokenizer": "classic",
          "filter": ["classic"]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /custom_classic_filter/_analyze
{
  "analyzer": "custom_classic",
  "text": "John's co-operate was excellent."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "John",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<APOSTROPHE>",
      "position": 0
    },
    {
      "token": "co",
      "start_offset": 7,
      "end_offset": 9,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "operate",
      "start_offset": 10,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "was",
      "start_offset": 18,
      "end_offset": 21,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "excellent",
      "start_offset": 22,
      "end_offset": 31,
      "type": "<ALPHANUM>",
      "position": 4
    }
  ]
}
```

