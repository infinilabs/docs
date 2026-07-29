---
title: "ASCII 折叠分词过滤器（ASCII Folding）"
date: 0001-01-01
summary: "ASCII Folding 分词过滤器 #  asciifolding 分词过滤器将非 ASCII 字符转换为与其最接近的 ASCII 等效字符。例如，&ldquo;é&rdquo; 变为 &ldquo;e&rdquo;，&ldquo;ü&rdquo; 变为 &ldquo;u&rdquo;，&ldquo;ñ&rdquo; 变为 &ldquo;n&rdquo;。这个过程被称为&quot;音译&quot;。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  ASCII 折叠分词过滤器有许多优点：
 增强搜索灵活性：用户在输入查询内容时常常会省略重音符号或特殊字符。ASCII 折叠分词过滤器可确保即使是这样的查询也仍然能返回相关结果。 规范化：通过确保重音字符始终被转换为其 ASCII 等效字符，使索引编制过程标准化。 国际化：对于包含多种语言和字符集的应用程序特别有用。   尽管 ASCII 折叠分词过滤器可以简化搜索，但它也可能导致特定信息的丢失，尤其是当数据集中重音字符和非重音字符之间的区别很重要的时候。
 参数说明 #  你可以使用 preserve_original 参数来配置 ASCII 折叠分词过滤器。将此参数设置为 true 时，会在词元流中同时保留原始词元及其 ASCII 折叠后的版本。当你希望在搜索查询中同时匹配一个词项的原始版本（带重音符号）和规范化版本（不带重音符号）时，这一点会特别有用。该参数的默认值为 false。
参考样例 #  以下示例请求创建了一个名为 example_index 的新索引，并定义了一个分词器，该分词器使用了 ASCII 折叠过滤器，且将 preserve_original 参数设置为 true：
PUT /example_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;custom_ascii_folding&#34;: { &#34;type&#34;: &#34;asciifolding&#34;, &#34;preserve_original&#34;: true } }, &#34;analyzer&#34;: { &#34;custom_ascii_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;custom_ascii_folding&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# ASCII Folding 分词过滤器

`asciifolding` 分词过滤器将非 ASCII 字符转换为与其最接近的 ASCII 等效字符。例如，"é" 变为 "e"，"ü" 变为 "u"，"ñ" 变为 "n"。这个过程被称为"音译"。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

ASCII 折叠分词过滤器有许多优点：

1. **增强搜索灵活性**：用户在输入查询内容时常常会省略重音符号或特殊字符。ASCII 折叠分词过滤器可确保即使是这样的查询也仍然能返回相关结果。
2. **规范化**：通过确保重音字符始终被转换为其 ASCII 等效字符，使索引编制过程标准化。
3. **国际化**：对于包含多种语言和字符集的应用程序特别有用。

> 尽管 ASCII 折叠分词过滤器可以简化搜索，但它也可能导致特定信息的丢失，尤其是当数据集中重音字符和非重音字符之间的区别很重要的时候。

## 参数说明

你可以使用 `preserve_original` 参数来配置 ASCII 折叠分词过滤器。将此参数设置为 `true` 时，会在词元流中同时保留原始词元及其 ASCII 折叠后的版本。当你希望在搜索查询中同时匹配一个词项的原始版本（带重音符号）和规范化版本（不带重音符号）时，这一点会特别有用。该参数的默认值为 `false`。

## 参考样例

以下示例请求创建了一个名为 `example_index` 的新索引，并定义了一个分词器，该分词器使用了 ASCII 折叠过滤器，且将 `preserve_original` 参数设置为 `true`：

```
PUT /example_index
{
  "settings": {
    "analysis": {
      "filter": {
        "custom_ascii_folding": {
          "type": "asciifolding",
          "preserve_original": true
        }
      },
      "analyzer": {
        "custom_ascii_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "custom_ascii_folding"
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
POST /example_index/_analyze
{
  "analyzer": "custom_ascii_analyzer",
  "text": "Résumé café naïve coördinate"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "resume",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "résumé",
      "start_offset": 0,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "cafe",
      "start_offset": 7,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "café",
      "start_offset": 7,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "naive",
      "start_offset": 12,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "naïve",
      "start_offset": 12,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "coordinate",
      "start_offset": 18,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "coördinate",
      "start_offset": 18,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 3
    }
  ]
}
```

