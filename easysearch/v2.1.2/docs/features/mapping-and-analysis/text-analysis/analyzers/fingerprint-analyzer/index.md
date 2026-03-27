---
title: "指纹分析器（Fingerprint）"
date: 0001-01-01
summary: "Fingerprint 分析器 #  fingerprint 分析器会创建一个文本指纹。该分析器对从输入生成的词项（词元）进行排序和去重，然后使用一个分隔符将它们连接起来。它通常用于数据去重，对于包含相同单词的相似输入，无论单词顺序如何，它都可以产生相同的输出结果。
fingerprint 分析器由以下组件组成：
 standard 分词器 lowercase 分词过滤器 asciifolding 分词过滤器 stop 分词过滤器 fingerprint 分词过滤器  相关指南（先读这些） #    文本分析基础  文本分析：规范化  参数说明 #  指纹分词器可以使用以下参数进行配置。
   参数 必填/可选 数据类型 描述     separator 可选 字符串 在对词项进行词元生成、排序和去重后，用于连接这些词项的字符。默认值为一个空格（ ）。   max_output_size 可选 整数 定义输出词元的最大大小。如果连接后的指纹超过此大小，将被截断。默认值为 255。   stopwords 可选 字符串或字符串列表 自定义的停用词列表。默认值为 _none_。   stopwords_path 可选 字符串 包含停用词列表的文件的路径（相对于配置目录的绝对路径或相对路径）。    参考样例 #  以下命令创建一个名为 my_custom_fingerprint_index 带有指纹分词器的索引："
---


# Fingerprint 分析器

`fingerprint` 分析器会创建一个文本指纹。该分析器对从输入生成的词项（词元）进行排序和去重，然后使用一个分隔符将它们连接起来。它通常用于数据去重，对于包含相同单词的相似输入，无论单词顺序如何，它都可以产生相同的输出结果。

`fingerprint` 分析器由以下组件组成：

1. `standard` 分词器
2. `lowercase` 分词过滤器
3. `asciifolding` 分词过滤器
4. `stop` 分词过滤器
5. `fingerprint` 分词过滤器

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

指纹分词器可以使用以下参数进行配置。

| 参数              | 必填/可选 | 数据类型           | 描述                                                                                  |
| ----------------- | --------- | ------------------ | ------------------------------------------------------------------------------------- |
| `separator`       | 可选      | 字符串             | 在对词项进行词元生成、排序和去重后，用于连接这些词项的字符。默认值为一个空格（` `）。 |
| `max_output_size` | 可选      | 整数               | 定义输出词元的最大大小。如果连接后的指纹超过此大小，将被截断。默认值为 `255`。        |
| `stopwords`       | 可选      | 字符串或字符串列表 | 自定义的停用词列表。默认值为 `_none_`。                                               |
| `stopwords_path`  | 可选      | 字符串             | 包含停用词列表的文件的路径（相对于配置目录的绝对路径或相对路径）。                    |

## 参考样例

以下命令创建一个名为 `my_custom_fingerprint_index` 带有指纹分词器的索引：

```
PUT /my_custom_fingerprint_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_fingerprint_analyzer": {
          "type": "fingerprint",
          "separator": "-",
          "max_output_size": 50,
          "stopwords": ["to", "the", "over", "and"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "my_custom_fingerprint_analyzer"
      }
    }
  }
}
```

## 产生的词元

以下请求用来检查使用该分词器生成的词元：

```
POST /my_custom_fingerprint_index/_analyze
{
  "analyzer": "my_custom_fingerprint_analyzer",
  "text": "The slow turtle swims over to the dog"
}
```

返回内容中包含了生成的词元：

```
{
  "tokens": [
    {
      "token": "dog-slow-swims-turtle",
      "start_offset": 0,
      "end_offset": 37,
      "type": "fingerprint",
      "position": 0
    }
  ]
}
```

## 深度定制

如果需要深度定制，你可以定义一个包含指纹分词器组件的分词器：

```
PUT /custom_fingerprint_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_fingerprint": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding",
            "fingerprint"
          ]
        }
      }
    }
  }
}
```

