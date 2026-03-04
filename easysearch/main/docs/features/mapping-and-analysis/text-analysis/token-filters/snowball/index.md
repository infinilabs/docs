---
title: "Snowball 词干分词过滤器（Snowball）"
date: 0001-01-01
summary: "Snowball 分词过滤器 #  snowball 分词过滤器是一种基于 雪球算法的词干提取过滤器。它支持多种语言，并且比波特词干提取算法更加高效和准确。
相关指南（先读这些） #    文本分析：词干提取  文本分析：规范化  参数说明 #  雪球分词过滤器可以通过一个 language（语言）参数进行配置，该参数接受以下值：
 阿拉伯语（Arabic） 亚美尼亚语（Armenian） 巴斯克语（Basque） 加泰罗尼亚语（Catalan） 丹麦语（Danish） 荷兰语（Dutch） 英语（English，默认值） 爱沙尼亚语（Estonian） 芬兰语（Finnish） 法语（French） 德语（German） 德语 2（German2） 匈牙利语（Hungarian） 意大利语（Italian） 爱尔兰语（Irish） Kp 立陶宛语（Lithuanian） 洛文斯（Lovins） 挪威语（Norwegian） 波特（Porter） 葡萄牙语（Portuguese） 罗马尼亚语（Romanian） 俄语（Russian） 西班牙语（Spanish） 瑞典语（Swedish） 土耳其语（Turkish）  参考样例 #  以下示例请求创建了一个名为 my-snowball-index 的新索引，并配置了一个带有雪球过滤器的分词器。
PUT /my-snowball-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_snowball_filter&#34;: { &#34;type&#34;: &#34;snowball&#34;, &#34;language&#34;: &#34;English&#34; } }, &#34;analyzer&#34;: { &#34;my_snowball_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;my_snowball_filter&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Snowball 分词过滤器

`snowball` 分词过滤器是一种基于[雪球算法](https://snowballstem.org/)的词干提取过滤器。它支持多种语言，并且比波特词干提取算法更加高效和准确。

## 相关指南（先读这些）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

雪球分词过滤器可以通过一个 `language`（语言）参数进行配置，该参数接受以下值：

- 阿拉伯语（`Arabic`）
- 亚美尼亚语（`Armenian`）
- 巴斯克语（`Basque`）
- 加泰罗尼亚语（`Catalan`）
- 丹麦语（`Danish`）
- 荷兰语（`Dutch`）
- 英语（`English`，默认值）
- 爱沙尼亚语（`Estonian`）
- 芬兰语（`Finnish`）
- 法语（`French`）
- 德语（`German`）
- 德语 2（`German2`）
- 匈牙利语（`Hungarian`）
- 意大利语（`Italian`）
- 爱尔兰语（`Irish`）
- `Kp`
- 立陶宛语（`Lithuanian`）
- 洛文斯（`Lovins`）
- 挪威语（`Norwegian`）
- 波特（`Porter`）
- 葡萄牙语（`Portuguese`）
- 罗马尼亚语（`Romanian`）
- 俄语（`Russian`）
- 西班牙语（`Spanish`）
- 瑞典语（`Swedish`）
- 土耳其语（`Turkish`）

## 参考样例

以下示例请求创建了一个名为 `my-snowball-index` 的新索引，并配置了一个带有雪球过滤器的分词器。

```
PUT /my-snowball-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_snowball_filter": {
          "type": "snowball",
          "language": "English"
        }
      },
      "analyzer": {
        "my_snowball_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_snowball_filter"
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
GET /my-snowball-index/_analyze
{
  "analyzer": "my_snowball_analyzer",
  "text": "running runners"
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
    }
  ]
}
```

