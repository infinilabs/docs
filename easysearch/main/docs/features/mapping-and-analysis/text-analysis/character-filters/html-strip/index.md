---
title: "HTML 标签字符过滤器（HTML Strip）"
date: 0001-01-01
summary: "HTML Strip 字符过滤器 #  html_strip 字符过滤器会从输入文本中移除 HTML 标签（例如 &lt;div&gt;、&lt;p&gt; 和 &lt;a&gt; 等）并输出纯文本。该过滤器可以配置保留某些标签，或者配置把特定的 HTML 标签实体（如 &amp;nbsp;）解码为空格。
相关指南（先读这些） #    文本分析基础  文本分析：识别词元  参考样例 #  以下请求展示将 html_strip 字符过滤器应用于文本：
GET /_analyze { &#34;tokenizer&#34;: &#34;keyword&#34;, &#34;char_filter&#34;: [ &#34;html_strip&#34; ], &#34;text&#34;: &#34;&lt;p&gt;Commonly used calculus symbols include &amp;alpha;, &amp;beta; and &amp;theta; &lt;/p&gt;&#34; } 返回内容中包含的词元里，可以看到 HTML 字符已被转换为它们的解码后的值：
{ &#34;tokens&#34;: [ { &#34;token&#34;: &#34;\nCommonly used calculus symbols include α, β and θ \n&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 74, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 0 } ] } 参数说明 #  html_strip 字符过滤器可以使用以下参数进行配置。"
---


# HTML Strip 字符过滤器

`html_strip` 字符过滤器会从输入文本中移除 HTML 标签（例如 `<div>`、`<p>` 和 `<a>` 等）并输出纯文本。该过滤器可以配置保留某些标签，或者配置把特定的 HTML 标签实体（如 `&nbsp;`）解码为空格。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下请求展示将 `html_strip` 字符过滤器应用于文本：

```
GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    "html_strip"
  ],
  "text": "<p>Commonly used calculus symbols include &alpha;, &beta; and &theta; </p>"
}
```

返回内容中包含的词元里，可以看到 HTML 字符已被转换为它们的解码后的值：

```
{
  "tokens": [
    {
      "token": "\nCommonly used calculus symbols include α, β and θ \n",
      "start_offset": 0,
      "end_offset": 74,
      "type": "word",
      "position": 0
    }
  ]
}
```

## 参数说明

`html_strip` 字符过滤器可以使用以下参数进行配置。

| 参数           | 必填/可选 | 数据类型   | 描述                                                                                                                                                                                       |
| -------------- | --------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `escaped_tags` | 可选      | 字符串数组 | 一个 HTML 元素名称列表，指定时不带包围的尖括号（`< >`）。当从文本中去除 HTML 标签时，过滤器不会移除该列表中的元素。例如，将该配置设置为 `["b", "i"]`时, 将防止 `<b>` 和 `<i>` 元素被去除。 |

### 示例：带有小写过滤器的自定义分词器

以下请求创建了一个自定义分词器，该分词器通过使用 `html_strip` 字符过滤器来去除 HTML 标签，并通过 `lowercase` 词元过滤器将纯文本转换为小写形式：

```
PUT /html_strip_and_lowercase_analyzer
{
  "settings": {
    "analysis": {
      "char_filter": {
        "html_filter": {
          "type": "html_strip"
        }
      },
      "analyzer": {
        "html_strip_analyzer": {
          "type": "custom",
          "char_filter": ["html_filter"],
          "tokenizer": "standard",
          "filter": ["lowercase"]
        }
      }
    }
  }
}
```

使用以下请求来检查使用该分词器生成的词元：

```
GET /html_strip_and_lowercase_analyzer/_analyze
{
  "analyzer": "html_strip_analyzer",
  "text": "<h1>Welcome to <strong>Easysearch</strong>!</h1>"
}
```

在返回内容中，HTML 标签已被移除，并且纯文本已被转换为小写形式：

```
{
  "tokens": [
    {
      "token": "welcome",
      "start_offset": 4,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "to",
      "start_offset": 12,
      "end_offset": 14,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "easysearch",
      "start_offset": 23,
      "end_offset": 42,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

### 示例：保留 HTML 标签的自定义分词器

以下示例请求创建了一个能保留 HTML 标签的自定义分词器：

```
PUT /html_strip_preserve_analyzer
{
  "settings": {
    "analysis": {
      "char_filter": {
        "html_filter": {
          "type": "html_strip",
          "escaped_tags": ["b", "i"]
        }
      },
      "analyzer": {
        "html_strip_analyzer": {
          "type": "custom",
          "char_filter": ["html_filter"],
          "tokenizer": "keyword"
        }
      }
    }
  }
}
```

使用以下请求来检查使用该分词器生成的词元：

```
GET /html_strip_preserve_analyzer/_analyze
{
  "analyzer": "html_strip_analyzer",
  "text": "<p>This is a <b>bold</b> and <i>italic</i> text.</p>"
}
```

在返回内容中，正如在自定义分词器请求中所指定的那样，斜体 `italic` 标签和加粗 `bold` 标签已被保留。

```
{
  "tokens": [
    {
      "token": "\nThis is a <b>bold</b> and <i>italic</i> text.\n",
      "start_offset": 0,
      "end_offset": 52,
      "type": "word",
      "position": 0
    }
  ]
}
```

