---
title: "字母分词器（Letter）"
date: 0001-01-01
summary: "Letter 分词器 #  letter 分词器会根据任何非字母字符将文本拆分成单词。它在处理许多欧洲语种时效果良好，但对于一些亚洲语种却不太有效，因为在亚洲语种中，单词之间不是由空格来分隔的。
相关指南（先读这些） #   文本分析：识别词元 文本分析基础  参考样例 #  下面的示例请求创建了一个名为 my_index 的新索引，并配置了一个使用字母词元生成器的分词器。
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_letter_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;letter&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_letter_analyzer&#34; } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST _analyze { &#34;tokenizer&#34;: &#34;letter&#34;, &#34;text&#34;: &#34;Cats 4EVER love chasing butterflies!&#34; } 返回内容包含产生的词元
{ &#34;tokens&#34;: [ { &#34;token&#34;: &#34;Cats&#34;, &#34;start_offset&#34;: 0, &#34;end_offset&#34;: 4, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 0 }, { &#34;token&#34;: &#34;EVER&#34;, &#34;start_offset&#34;: 6, &#34;end_offset&#34;: 10, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 1 }, { &#34;token&#34;: &#34;love&#34;, &#34;start_offset&#34;: 11, &#34;end_offset&#34;: 15, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 2 }, { &#34;token&#34;: &#34;chasing&#34;, &#34;start_offset&#34;: 16, &#34;end_offset&#34;: 23, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 3 }, { &#34;token&#34;: &#34;butterflies&#34;, &#34;start_offset&#34;: 24, &#34;end_offset&#34;: 35, &#34;type&#34;: &#34;word&#34;, &#34;position&#34;: 4 } ] } "
---


# Letter 分词器

`letter` 分词器会根据任何非字母字符将文本拆分成单词。它在处理许多欧洲语种时效果良好，但对于一些亚洲语种却不太有效，因为在亚洲语种中，单词之间不是由空格来分隔的。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

下面的示例请求创建了一个名为 `my_index` 的新索引，并配置了一个使用字母词元生成器的分词器。

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_letter_analyzer": {
          "type": "custom",
          "tokenizer": "letter"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_letter_analyzer"
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST _analyze
{
  "tokenizer": "letter",
  "text": "Cats 4EVER love chasing butterflies!"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "Cats",
      "start_offset": 0,
      "end_offset": 4,
      "type": "word",
      "position": 0
    },
    {
      "token": "EVER",
      "start_offset": 6,
      "end_offset": 10,
      "type": "word",
      "position": 1
    },
    {
      "token": "love",
      "start_offset": 11,
      "end_offset": 15,
      "type": "word",
      "position": 2
    },
    {
      "token": "chasing",
      "start_offset": 16,
      "end_offset": 23,
      "type": "word",
      "position": 3
    },
    {
      "token": "butterflies",
      "start_offset": 24,
      "end_offset": 35,
      "type": "word",
      "position": 4
    }
  ]
}
```

