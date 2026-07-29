---
title: "标准分析器（Standard）"
date: 0001-01-01
summary: "Standard 分析器 #  standard 分析器是在未指定其他分析器时默认使用的分析器。它旨在为通用文本处理提供一种基础且高效的方法。
该分析器由以下分词器和分词过滤器组成：
 standard 分词器：去除大部分标点符号，并依据空格和其他常见分隔符对文本进行分割。 lowercase 分词过滤器：将所有词元转换为小写，以确保匹配时不区分大小写。 stop 分词过滤器：从分词后的输出中移除常见的停用词，例如 &ldquo;the&rdquo;、&ldquo;is&rdquo; 和 &ldquo;and&rdquo;。  相关指南（先读这些） #    文本分析基础  文本分析：识别词元  参考样例 #  以下命令创建一个名为 my_standard_index 并使用标准分词器的索引：
PUT /my_standard_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;standard&#34; } } } } 参数说明 #  你可以使用以下参数来配置标准分词器。
   参数 必填/可选 数据类型 描述     max_token_length 可选 整数 设置生成的词元的最大长度。如果超过此长度，词元将按照 max_token_length 中配置的长度拆分为多个词元。默认值为 255。   stopwords 可选 字符串或字符串列表 一个包含自定义停用词列表（如 _english）的字符串，或者一个自定义停用词列表的数组。默认值为 _none_。   stopwords_path 可选 字符串 包含停用词列表的文件的路径（相对于配置目录的绝对路径或相对路径）。    配置自定义分词器 #  以下命令配置一个索引，该索引带有一个等同于标准分词器的自定义分词器："
---


# Standard 分析器

`standard` 分析器是在未指定其他分析器时默认使用的分析器。它旨在为通用文本处理提供一种基础且高效的方法。

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：去除大部分标点符号，并依据空格和其他常见分隔符对文本进行分割。
- `lowercase` 分词过滤器：将所有词元转换为小写，以确保匹配时不区分大小写。
- `stop` 分词过滤器：从分词后的输出中移除常见的停用词，例如 "the"、"is" 和 "and"。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参考样例

以下命令创建一个名为 `my_standard_index` 并使用标准分词器的索引：

```
PUT /my_standard_index
{
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "standard"
      }
    }
  }
}
```

## 参数说明

你可以使用以下参数来配置标准分词器。

| 参数               | 必填/可选 | 数据类型           | 描述                                                                                                                 |
| ------------------ | --------- | ------------------ | -------------------------------------------------------------------------------------------------------------------- |
| `max_token_length` | 可选      | 整数               | 设置生成的词元的最大长度。如果超过此长度，词元将按照 `max_token_length` 中配置的长度拆分为多个词元。默认值为 `255`。 |
| `stopwords`        | 可选      | 字符串或字符串列表 | 一个包含自定义停用词列表（如 `_english`）的字符串，或者一个自定义停用词列表的数组。默认值为 `_none_`。               |
| `stopwords_path`   | 可选      | 字符串             | 包含停用词列表的文件的路径（相对于配置目录的绝对路径或相对路径）。                                                   |

## 配置自定义分词器

以下命令配置一个索引，该索引带有一个等同于标准分词器的自定义分词器：

```
PUT /my_custom_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop"
          ]
        }
      }
    }
  }
}
```

## 产生的词元

以下请求用来检查分词器生成的词元：

```
POST /my_custom_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "The slow turtle swims away"
}
```

返回内容中包含了产生的词元

```
{
  "tokens": [
    {"token": "slow","start_offset": 4,"end_offset": 8,"type": "<ALPHANUM>","position": 1},
    {"token": "turtle","start_offset": 9,"end_offset": 15,"type": "<ALPHANUM>","position": 2},
    {"token": "swims","start_offset": 16,"end_offset": 21,"type": "<ALPHANUM>","position": 3},
    {"token": "away","start_offset": 22,"end_offset": 26,"type": "<ALPHANUM>","position": 4}
  ]
}
```

