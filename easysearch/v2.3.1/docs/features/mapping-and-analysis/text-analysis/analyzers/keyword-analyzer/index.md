---
title: "关键字分析器（Keyword）"
date: 0001-01-01
summary: "Keyword 分析器 #  keyword 分析器根本不会对文本进行分词。相反，它将整个输入视为单个词元，不会将其拆分成单个的词项。keyword 分析器常用于包含电子邮件地址、网址或产品 ID 的字段，以及其他不需要进行分词的情况。
相关指南（先读这些） #    文本分析基础  文本分析：识别词元  参考样例 #  以下命令创建一个名为 my_keyword_index 使用关键词分词器的索引：
PUT /my_keyword_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;keyword&#34; } } } } 配置自定义分词器 #  以下命令来配置一个自定义分词器的索引，该自定义分词器的作用等同于关键词分词器：
PUT /my_custom_keyword_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_keyword_analyzer&#34;: { &#34;tokenizer&#34;: &#34;keyword&#34; } } } } } 产生的词元 #  以下请求来检查使用该分词器生成的词元：
POST /my_custom_keyword_index/_analyze { &#34;analyzer&#34;: &#34;my_keyword_analyzer&#34;, &#34;text&#34;: &#34;Just one token&#34; } 返回内容包含产生的词元"
---


# Keyword 分析器

`keyword` 分析器根本不会对文本进行分词。相反，它将整个输入视为单个词元，不会将其拆分成单个的词项。`keyword` 分析器常用于包含电子邮件地址、网址或产品 ID 的字段，以及其他不需要进行分词的情况。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下命令创建一个名为 `my_keyword_index` 使用关键词分词器的索引：

```
PUT /my_keyword_index
{
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "keyword"
      }
    }
  }
}
```

## 配置自定义分词器

以下命令来配置一个自定义分词器的索引，该自定义分词器的作用等同于关键词分词器：

```
PUT /my_custom_keyword_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_keyword_analyzer": {
          "tokenizer": "keyword"
        }
      }
    }
  }
}
```

## 产生的词元

以下请求来检查使用该分词器生成的词元：

```
POST /my_custom_keyword_index/_analyze
{
  "analyzer": "my_keyword_analyzer",
  "text": "Just one token"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "Just one token",
      "start_offset": 0,
      "end_offset": 14,
      "type": "word",
      "position": 0
    }
  ]
}
```

