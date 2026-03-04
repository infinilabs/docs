---
title: "空白分析器（Whitespace）"
date: 0001-01-01
summary: "Whitespace 分析器 #  whitespace 分析器仅基于空白字符（例如，空格和制表符）将文本拆分为词元。比如转换为小写形式或移除停用词这样的转换操作，它都不会应用，因此文本的原始大小写形式会被保留，并且标点符号也会作为词元的一部分包含在内。
相关指南（先读这些） #    文本分析基础  文本分析：识别词元  参考样例 #  以下命令创建一个名为 my_whitespace_index 并使用空格分词器的索引：
PUT /my_whitespace_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;whitespace&#34; } } } } 配置自定义分词器 #  以下命令来配置一个自定义分词器的索引，该自定义分词器的作用等同于空格分词器：
PUT /my_custom_whitespace_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_custom_whitespace_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;whitespace&#34;, &#34;filter&#34;: [&#34;lowercase&#34;] } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_custom_whitespace_analyzer&#34; } } } } 产生的词元 #  以下请求用来检查分词器生成的词元："
---


# Whitespace 分析器

`whitespace` 分析器仅基于空白字符（例如，空格和制表符）将文本拆分为词元。比如转换为小写形式或移除停用词这样的转换操作，它都不会应用，因此文本的原始大小写形式会被保留，并且标点符号也会作为词元的一部分包含在内。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下命令创建一个名为 `my_whitespace_index` 并使用空格分词器的索引：

```
PUT /my_whitespace_index
{
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "whitespace"
      }
    }
  }
}
```

## 配置自定义分词器

以下命令来配置一个自定义分词器的索引，该自定义分词器的作用等同于空格分词器：

```
PUT /my_custom_whitespace_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_whitespace_analyzer": {
          "type": "custom",
          "tokenizer": "whitespace",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "my_custom_whitespace_analyzer"
      }
    }
  }
}
```

## 产生的词元

以下请求用来检查分词器生成的词元：

```
POST /my_custom_whitespace_index/_analyze
{
  "analyzer": "my_custom_whitespace_analyzer",
  "text": "The SLOW turtle swims away! 123"
}
```

返回内容中包含了产生的词元

```
{
  "tokens": [
    {"token": "the","start_offset": 0,"end_offset": 3,"type": "word","position": 0},
    {"token": "slow","start_offset": 4,"end_offset": 8,"type": "word","position": 1},
    {"token": "turtle","start_offset": 9,"end_offset": 15,"type": "word","position": 2},
    {"token": "swims","start_offset": 16,"end_offset": 21,"type": "word","position": 3},
    {"token": "away!","start_offset": 22,"end_offset": 27,"type": "word","position": 4},
    {"token": "123","start_offset": 28,"end_offset": 31,"type": "word","position": 5}
  ]
}
```

