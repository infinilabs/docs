---
title: "Porter 词干分词过滤器（Porter Stem）"
date: 0001-01-01
summary: "Porter Stem 分词过滤器 #  porter_stem 分词过滤器会将单词还原为其基本（或词干）形式，并去除单词中常见的后缀，这有助于通过单词的词根来匹配相似的单词。例如，单词&quot;running&quot;会被词干提取为&quot;run&quot;。此分词过滤器主要用于英语，并基于 波特词干提取算法进行词干提取操作。
相关指南（先读这些） #    文本分析：词干提取  文本分析：规范化  参考样例 #  以下示例请求创建了一个名为 my_stem_index 的新索引，并配置了一个带有波特词干提取过滤器的分词器。
PUT /my_stem_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_porter_stem&#34;: { &#34;type&#34;: &#34;porter_stem&#34; } }, &#34;analyzer&#34;: { &#34;porter_analyzer&#34;: { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;my_porter_stem&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /my_stem_index/_analyze { &#34;text&#34;: &#34;running runners ran&#34;, &#34;analyzer&#34;: &#34;porter_analyzer&#34; } 返回内容包含产生的词元
{ &#34;tokens&#34;: [ { &#34;token&#34;: &#34;run&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 7, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 0 }, { &#34;token&#34;: &#34;runner&#34;, &#34;start_offset&#34;: 8, &#34;end_offset&#34;: 15, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 1 }, { &#34;token&#34;: &#34;ran&#34;, &#34;start_offset&#34;: 16, &#34;end_offset&#34;: 19, &#34;type&#34;: &#34;&lt;ALPHANUM&gt;&#34;, &#34;position&#34;: 2 } ] } "
---


# Porter Stem 分词过滤器

`porter_stem` 分词过滤器会将单词还原为其基本（或词干）形式，并去除单词中常见的后缀，这有助于通过单词的词根来匹配相似的单词。例如，单词"running"会被词干提取为"run"。此分词过滤器主要用于英语，并基于[波特词干提取算法](https://snowballstem.org/algorithms/porter/stemmer.html)进行词干提取操作。

## 相关指南（先读这些）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参考样例

以下示例请求创建了一个名为 `my_stem_index` 的新索引，并配置了一个带有波特词干提取过滤器的分词器。

```
PUT /my_stem_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_porter_stem": {
          "type": "porter_stem"
        }
      },
      "analyzer": {
        "porter_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_porter_stem"
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
POST /my_stem_index/_analyze
{
  "text": "running runners ran",
  "analyzer": "porter_analyzer"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "run",
      "start_offset": 0,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "runner",
      "start_offset": 8,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "ran",
      "start_offset": 16,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

