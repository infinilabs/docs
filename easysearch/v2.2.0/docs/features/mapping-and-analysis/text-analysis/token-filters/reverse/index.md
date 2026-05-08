---
title: "反转分词过滤器（Reverse）"
date: 0001-01-01
summary: "Reverse 分词过滤器 #  reverse 分词过滤器会反转每个词元中字符的顺序，这样在分析过程中，后缀信息就会位于反转后词元的开头。
相关指南（先读这些） #    文本分析：识别词元  部分匹配  文本分析基础  这对于基于后缀的搜索很有用：
反转分词过滤器在你需要进行基于后缀的搜索时很有帮助，例如以下场景：
 后缀匹配：根据单词的后缀来搜索单词，比如识别以特定后缀结尾的单词（例如 -tion 或 -ing）。 文件扩展名搜索：通过文件的扩展名来搜索文件，例如 .txt 或 .jpg。 自定义排序或排名：通过反转词元，你可以基于后缀实现独特的排序或排名逻辑。 后缀自动补全：实现基于后缀而非前缀的自动补全建议。  参考说明 #  以下示例请求创建了一个名为 my-reverse-index 的新索引，并配置了一个带有反转过滤器的分词器。
PUT /my-reverse-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;reverse_filter&#34;: { &#34;type&#34;: &#34;reverse&#34; } }, &#34;analyzer&#34;: { &#34;my_reverse_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;reverse_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Reverse 分词过滤器

`reverse` 分词过滤器会反转每个词元中字符的顺序，这样在分析过程中，后缀信息就会位于反转后词元的开头。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

这对于基于后缀的搜索很有用：

反转分词过滤器在你需要进行基于后缀的搜索时很有帮助，例如以下场景：

- **后缀匹配**：根据单词的后缀来搜索单词，比如识别以特定后缀结尾的单词（例如 -tion 或 -ing）。
- **文件扩展名搜索**：通过文件的扩展名来搜索文件，例如 .txt 或 .jpg。
- **自定义排序或排名**：通过反转词元，你可以基于后缀实现独特的排序或排名逻辑。
- **后缀自动补全**：实现基于后缀而非前缀的自动补全建议。

## 参考说明

以下示例请求创建了一个名为 `my-reverse-index` 的新索引，并配置了一个带有反转过滤器的分词器。

```
PUT /my-reverse-index
{
  "settings": {
    "analysis": {
      "filter": {
        "reverse_filter": {
          "type": "reverse"
        }
      },
      "analyzer": {
        "my_reverse_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "reverse_filter"
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
GET /my-reverse-index/_analyze
{
  "analyzer": "my_reverse_analyzer",
  "text": "hello world"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "olleh",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "dlrow",
      "start_offset": 6,
      "end_offset": 11,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

