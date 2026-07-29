---
title: "停用词分析器（Stop）"
date: 0001-01-01
summary: "Stop 分析器 #  stop 分析器会在文本中移除预定义的停用词。该分析器由一个小写分词器和一个停用词分词过滤器组成。
相关指南（先读这些） #    文本分析基础  文本分析：停用词  文本分析：识别词元  参数说明 #  你可以使用以下参数来配置一个停用词分词器。
   参数 必填/可选 数据类型 描述     stopwords 可选 字符串或字符串列表 一个包含自定义停用词列表（如 _english）的字符串，或者一个自定义停用词列表的数组。默认值为 _none_。   stopwords_path 可选 字符串 包含停用词列表的文件的路径（相对于配置目录的绝对路径或相对路径）。    参考样例 #  以下命令创建一个名为 my_stop_index 并使用停词器分词器的索引：
PUT /my_stop_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;stop&#34; } } } } 配置自定义分词器 #  以下命令来配置一个自定义分词器的索引，该自定义分词器的作用等同于停用词分词器："
---


# Stop 分析器

`stop` 分析器会在文本中移除预定义的停用词。该分析器由一个小写分词器和一个停用词分词过滤器组成。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：停用词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stopwords.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## 参数说明

你可以使用以下参数来配置一个停用词分词器。

| 参数             | 必填/可选 | 数据类型           | 描述                                                                                                   |
| ---------------- | --------- | ------------------ | ------------------------------------------------------------------------------------------------------ |
| `stopwords`      | 可选      | 字符串或字符串列表 | 一个包含自定义停用词列表（如 `_english`）的字符串，或者一个自定义停用词列表的数组。默认值为 `_none_`。 |
| `stopwords_path` | 可选      | 字符串             | 包含停用词列表的文件的路径（相对于配置目录的绝对路径或相对路径）。                                     |

## 参考样例

以下命令创建一个名为 `my_stop_index` 并使用停词器分词器的索引：

```
PUT /my_stop_index
{
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "stop"
      }
    }
  }
}
```

## 配置自定义分词器

以下命令来配置一个自定义分词器的索引，该自定义分词器的作用等同于停用词分词器：

```
PUT /my_custom_stop_analyzer_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_stop_analyzer": {
          "tokenizer": "lowercase",
          "filter": [
            "stop"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "my_custom_stop_analyzer"
      }
    }
  }
}
```

## 产生的词元

以下请求用来检查分词器生成的词元：

```
POST /my_custom_stop_analyzer_index/_analyze
{
  "analyzer": "my_custom_stop_analyzer",
  "text": "The large turtle is green and brown"
}
```

返回内容中包含了产生的词元

```
{
  "tokens": [
    {
      "token": "large",
      "start_offset": 4,
      "end_offset": 9,
      "type": "word",
      "position": 1
    },
    {
      "token": "turtle",
      "start_offset": 10,
      "end_offset": 16,
      "type": "word",
      "position": 2
    },
    {
      "token": "green",
      "start_offset": 20,
      "end_offset": 25,
      "type": "word",
      "position": 4
    },
    {
      "token": "brown",
      "start_offset": 30,
      "end_offset": 35,
      "type": "word",
      "position": 6
    }
  ]
}
```

## 指定停用词

下面演示了怎么去指定一个自定义的停用词列表：

```
PUT /my_new_custom_stop_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_stop_analyzer": {
          "type": "stop",
          "stopwords_path": "stopwords.txt"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "my_custom_stop_analyzer"
      }
    }
  }
}
```

在这个例子中，该文件位于配置目录中。你也可以指定该文件的完整路径。

