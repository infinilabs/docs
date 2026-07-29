---
title: "关键字分词器（Keyword）"
date: 0001-01-01
summary: "Keyword 分词器 #  keyword 分词器接收文本并将其原封不动地作为单个词元输出。当你希望输入内容保持完整时，比如在管理像姓名、产品代码或电子邮件地址这类结构化数据时，这个分词器就特别有用。
keyword 分词器可以与词元过滤器搭配使用来处理文本。例如，对文本进行规范化处理或去除多余的字符。
相关指南（先读这些） #    文本分析：识别词元  文本分析基础  参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个使用关键字词元生成器的分词器：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_keyword_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;keyword&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;my_keyword_analyzer&#34; } } } } 生成的词元 #  使用以下请求来检查使用该分词器生成的词元：
POST /my_index/_analyze { &#34;analyzer&#34;: &#34;my_keyword_analyzer&#34;, &#34;text&#34;: &#34;Easysearch Example&#34; } 返回内容会是包含原始内容的单个词元："
---


# Keyword 分词器

`keyword` 分词器接收文本并将其原封不动地作为单个词元输出。当你希望输入内容保持完整时，比如在管理像姓名、产品代码或电子邮件地址这类结构化数据时，这个分词器就特别有用。

`keyword` 分词器可以与词元过滤器搭配使用来处理文本。例如，对文本进行规范化处理或去除多余的字符。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个使用关键字词元生成器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_keyword_analyzer": {
          "type": "custom",
          "tokenizer": "keyword"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_keyword_analyzer"
      }
    }
  }
}
```

## 生成的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index/_analyze
{
  "analyzer": "my_keyword_analyzer",
  "text": "Easysearch Example"
}
```

返回内容会是包含原始内容的单个词元：

```
{
  "tokens": [
    {
      "token": "Easysearch Example",
      "start_offset": 0,
      "end_offset": 18,
      "type": "word",
      "position": 0
    }
  ]
}
```

## 参数说明

关键字词元生成器可以使用以下参数进行配置。

| 参数          | 必需/可选 | 数据类型 | 描述                                                     |
| ------------- | --------- | -------- | -------------------------------------------------------- |
| `buffer_size` | 可选      | 整数     | 确定字符缓冲区的大小。默认值为 256。通常无需更改此设置。 |

## 将关键字词元生成器与词元过滤器结合使用

为了增强关键字词元生成器的功能，你可以将它与词元过滤器结合起来。词元过滤器能够对文本进行转换，例如将文本转换为小写形式或者移除不需要的字符。

### 示例：使用匹配替换（pattern_replace）词元过滤器和关键字词元生成器

在这个示例中，匹配替换（`pattern_replace`）词元过滤器使用正则表达式，会将所有非字母数字字符替换为空字符串：

```
POST _analyze
{
  "tokenizer": "keyword",
  "filter": [
    {
      "type": "pattern_replace",
      "pattern": "[^a-zA-Z0-9]",
      "replacement": ""
    }
  ],
  "text": "Product#1234-XYZ"
}
```

`pattern_replace`词元过滤器会移除非字母数字字符，并返回以下词元：

```
{
  "tokens": [
    {
      "token": "Product1234XYZ",
      "start_offset": 0,
      "end_offset": 16,
      "type": "word",
      "position": 0
    }
  ]
}
```

