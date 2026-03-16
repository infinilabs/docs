---
title: "展平图分词过滤器（Flatten Graph）"
date: 0001-01-01
summary: "Flatten Graph 分词过滤器 #  flatten_graph 分词过滤器用于处理在图结构中同一位置生成多个词元时出现的复杂词元关系。一些分词过滤器，比如 synonym_graph 和 word_delimiter_graph，会生成多位置词元——即相互重叠或跨越多个位置的词元。这些词元图对于搜索查询很有用，但在索引过程中却不被直接支持。flatten_graph 分词过滤器会将多位置词元解析为一个线性的词元序列。对图进行扁平化处理可确保与索引过程兼容。
 注意：词元图的扁平化是一个有损耗的过程。只要有可能，就应避免使用 flatten_graph 过滤器。相反，应仅在搜索分词器中应用图分词过滤器，这样就无需使用 flatten_graph 分词过滤器了。
 相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参考样例 #  以下示例请求创建了一个名为 test_index 的新索引，并配置了一个带有平图化分词过滤器的分词器：
PUT /test_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_index_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;my_custom_filter&#34;, &#34;flatten_graph&#34; ] } }, &#34;filter&#34;: { &#34;my_custom_filter&#34;: { &#34;type&#34;: &#34;word_delimiter_graph&#34;, &#34;catenate_all&#34;: true } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Flatten Graph 分词过滤器

`flatten_graph` 分词过滤器用于处理在图结构中同一位置生成多个词元时出现的复杂词元关系。一些分词过滤器，比如 `synonym_graph` 和 `word_delimiter_graph`，会生成多位置词元——即相互重叠或跨越多个位置的词元。这些词元图对于搜索查询很有用，但在索引过程中却不被直接支持。`flatten_graph` 分词过滤器会将多位置词元解析为一个线性的词元序列。对图进行扁平化处理可确保与索引过程兼容。

> **注意**：词元图的扁平化是一个有损耗的过程。只要有可能，就应避免使用 `flatten_graph` 过滤器。相反，应仅在搜索分词器中应用图分词过滤器，这样就无需使用 `flatten_graph` 分词过滤器了。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参考样例

以下示例请求创建了一个名为 `test_index` 的新索引，并配置了一个带有平图化分词过滤器的分词器：

```
PUT /test_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_index_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "my_custom_filter",
            "flatten_graph"
          ]
        }
      },
      "filter": {
        "my_custom_filter": {
          "type": "word_delimiter_graph",
          "catenate_all": true
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /test_index/_analyze
{
  "analyzer": "my_index_analyzer",
  "text": "Easysearch helped many employers"
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
      "token": "helped",
      "start_offset": 11,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "many",
      "start_offset": 18,
      "end_offset": 22,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "employers",
      "start_offset": 23,
      "end_offset": 32,
      "type": "<ALPHANUM>",
      "position": 3
    }
  ]
}
```

