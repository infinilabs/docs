---
title: "正则表达式分析器（Pattern）"
date: 0001-01-01
summary: "Pattern 分析器 #  pattern 分析器允许你定义一个自定义分析器，该分析器使用正则表达式（regex）将输入文本分割成词元。它还提供了应用正则表达式、将词元转换为小写以及过滤掉停用词的功能。
相关指南（先读这些） #    文本分析基础  文本分析：识别词元  参数说明 #  匹配模式分词器可以使用以下参数进行配置。
   参数 必填/可选 数据类型 描述     pattern 可选 字符串 用于对输入内容进行分词匹配的 Java 正则表达式。默认值是 \W+。   flags 可选 字符串 以竖线分隔的 Java 正则表达式的字符串标志，这些标志会修改正则表达式的行为。   lowercase 可选 布尔值 是否将词元转换为小写。默认值为 true。   stopwords 可选 字符串或字符串列表 个包含自定义停用词列表（如 _english）的字符串，或者一个自定义停用词列表的数组。默认值为 _none_。   stopwords_path 可选 字符串 包含停用词列表的文件的路径（相对于配置目录的绝对路径或相对路径）。    参考样例 #  以下命令创建一个名为 my_pattern_index 并使用匹配模式分词器的索引："
---


# Pattern 分析器

`pattern` 分析器允许你定义一个自定义分析器，该分析器使用正则表达式（regex）将输入文本分割成词元。它还提供了应用正则表达式、将词元转换为小写以及过滤掉停用词的功能。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

匹配模式分词器可以使用以下参数进行配置。

| 参数             | 必填/可选 | 数据类型           | 描述                                                                                                 |
| ---------------- | --------- | ------------------ | ---------------------------------------------------------------------------------------------------- |
| `pattern`        | 可选      | 字符串             | 用于对输入内容进行分词匹配的 Java 正则表达式。默认值是 `\W+`。                                       |
| `flags`          | 可选      | 字符串             | 以竖线分隔的 Java 正则表达式的字符串标志，这些标志会修改正则表达式的行为。                           |
| `lowercase`      | 可选      | 布尔值             | 是否将词元转换为小写。默认值为 `true`。                                                              |
| `stopwords`      | 可选      | 字符串或字符串列表 | 个包含自定义停用词列表（如 `_english`）的字符串，或者一个自定义停用词列表的数组。默认值为 `_none_`。 |
| `stopwords_path` | 可选      | 字符串             | 包含停用词列表的文件的路径（相对于配置目录的绝对路径或相对路径）。                                   |

## 参考样例

以下命令创建一个名为 `my_pattern_index` 并使用匹配模式分词器的索引：

```
PUT /my_pattern_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_pattern_analyzer": {
          "type": "pattern",
          "pattern": "\\W+",
          "lowercase": true,
          "stopwords": ["and", "is"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "my_pattern_analyzer"
      }
    }
  }
}
```

## 产生的词元

以下请求用来检查分词器生成的词元：

```
POST /my_pattern_index/_analyze
{
  "analyzer": "my_pattern_analyzer",
  "text": "Easysearch is fast and scalable"
}
```

返回内容中包含了产生的词元

```
{
  "tokens": [
    {
      "token": "easysearch",
      "start_offset": 0,
      "end_offset": 10,
      "type": "word",
      "position": 0
    },
    {
      "token": "fast",
      "start_offset": 14,
      "end_offset": 18,
      "type": "word",
      "position": 2
    },
    {
      "token": "scalable",
      "start_offset": 23,
      "end_offset": 31,
      "type": "word",
      "position": 4
    }
  ]
}
```

