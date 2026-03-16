---
title: "简单模式分词器（Simple Pattern）"
date: 0001-01-01
summary: "Simple Pattern 分词器 #  simple_pattern 分词器使用正则表达式匹配文本，将匹配到的内容作为词元输出。它与 simple_pattern_split 的区别在于：simple_pattern 输出匹配的部分，而 simple_pattern_split 输出被分隔的部分。
该分词器使用 Lucene 正则表达式，语法是标准正则表达式的子集。
参数 #     参数 说明 默认值     pattern Lucene 正则表达式模式 空字符串（匹配空串）    示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_tokenizer&#34;: { &#34;type&#34;: &#34;simple_pattern&#34;, &#34;pattern&#34;: &#34;[0-9]{3}&#34; } }, &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;tokenizer&#34;: &#34;my_tokenizer&#34; } } } } } POST my_index/_analyze { &#34;analyzer&#34;: &#34;my_analyzer&#34;, &#34;text&#34;: &#34;fd]]]-%]afd][ 123 fd-ede 456&#34; } 以上示例将产生 123 和 456 两个词元。"
---


# Simple Pattern 分词器

`simple_pattern` 分词器使用正则表达式匹配文本，将**匹配到的内容**作为词元输出。它与 `simple_pattern_split` 的区别在于：`simple_pattern` 输出**匹配的部分**，而 `simple_pattern_split` 输出**被分隔的部分**。

该分词器使用 [Lucene 正则表达式](https://lucene.apache.org/core/8_7_0/core/org/apache/lucene/util/automaton/RegExp.html)，语法是标准正则表达式的子集。

## 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `pattern` | Lucene 正则表达式模式 | 空字符串（匹配空串） |

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_tokenizer": {
          "type": "simple_pattern",
          "pattern": "[0-9]{3}"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      }
    }
  }
}

POST my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "fd]]]-%]afd][ 123 fd-ede 456"
}
```

以上示例将产生 `123` 和 `456` 两个词元。

## 相关指南

- [Simple Pattern Split 分词器]({{< relref "./simple-pattern-split" >}})
- [Pattern 分词器]({{< relref "./pattern" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

