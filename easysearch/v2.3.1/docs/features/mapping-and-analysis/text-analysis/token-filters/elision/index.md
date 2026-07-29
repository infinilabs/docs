---
title: "省略词分词过滤器（Elision）"
date: 0001-01-01
summary: "Elision 分词过滤器 #  elision 分词过滤器用于从某些语言的单词中去除省略的字符。省略现象通常出现在像法语这样的语言中，在这些语言里，单词常常会发生缩合，并与后面的单词结合，常见的方式是省略一个元音字母，并用一个撇号来替代。
 注意：elision 分词过滤器已经在以下语言分词器中预先配置好了：加泰罗尼亚语（catalan）、法语（french）、爱尔兰语（irish）和意大利语（italian）。
 相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  参数说明 #  自定义省略词分词过滤器可使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     articles 若未配置 articles_path 则为必需 字符串数组 定义当某些冠词或短词作为省略形式的一部分出现时，哪些应该被移除。   articles_path 若未配置 articles 则为必需 字符串 指定在分析过程中应被移除的自定义冠词列表的路径。   articles_case 可选 布尔值 指定在匹配省略形式时，该过滤器是否区分大小写。默认值为 false。    参考样例 #  法语中默认的省略形式集合包括 l'、m'、t'、qu'、n'、s'、j'、d'、c'、jusqu'、quoiqu'、lorsqu' 和 puisqu'。你可以通过配置 french_elision 分词过滤器来更新这个集合。以下示例请求创建了一个名为 french_texts 的新索引，并配置了一个带有 french_elision 过滤器的分词器："
---


# Elision 分词过滤器

`elision` 分词过滤器用于从某些语言的单词中去除省略的字符。省略现象通常出现在像法语这样的语言中，在这些语言里，单词常常会发生缩合，并与后面的单词结合，常见的方式是省略一个元音字母，并用一个撇号来替代。

> **注意**：`elision` 分词过滤器已经在以下语言分词器中预先配置好了：加泰罗尼亚语（`catalan`）、法语（`french`）、爱尔兰语（`irish`）和意大利语（`italian`）。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

自定义省略词分词过滤器可使用以下参数进行配置。

| 参数            | 必需/可选                         | 数据类型   | 描述                                                             |
| --------------- | --------------------------------- | ---------- | ---------------------------------------------------------------- |
| `articles`      | 若未配置 `articles_path` 则为必需 | 字符串数组 | 定义当某些冠词或短词作为省略形式的一部分出现时，哪些应该被移除。 |
| `articles_path` | 若未配置 `articles` 则为必需      | 字符串     | 指定在分析过程中应被移除的自定义冠词列表的路径。                 |
| `articles_case` | 可选                              | 布尔值     | 指定在匹配省略形式时，该过滤器是否区分大小写。默认值为 `false`。 |

## 参考样例

法语中默认的省略形式集合包括 `l'`、`m'`、`t'`、`qu'`、`n'`、`s'`、`j'`、`d'`、`c'`、`jusqu'`、`quoiqu'`、`lorsqu'` 和 `puisqu'`。你可以通过配置 `french_elision` 分词过滤器来更新这个集合。以下示例请求创建了一个名为 `french_texts` 的新索引，并配置了一个带有 `french_elision` 过滤器的分词器：

```
PUT /french_texts
{
  "settings": {
    "analysis": {
      "filter": {
        "french_elision": {
          "type": "elision",
          "articles": [ "l", "t", "m", "d", "n", "s", "j" ]
        }
      },
      "analyzer": {
        "french_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "french_elision"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "french_analyzer"
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /french_texts/_analyze
{
  "analyzer": "french_analyzer",
  "text": "L'étudiant aime l'école et le travail."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "étudiant",
      "start_offset": 0,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "aime",
      "start_offset": 11,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "école",
      "start_offset": 16,
      "end_offset": 23,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "et",
      "start_offset": 24,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "le",
      "start_offset": 27,
      "end_offset": 29,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "travail",
      "start_offset": 30,
      "end_offset": 37,
      "type": "<ALPHANUM>",
      "position": 5
    }
  ]
}
```

