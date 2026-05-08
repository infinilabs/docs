---
title: "同义词分词过滤器（Synonym）"
date: 0001-01-01
summary: "Synonym 分词过滤器 #  synonym 分词过滤器允许将多个术语映射到单个术语，或在单词之间创建等价组，从而提高搜索的灵活性。
相关指南（先读这些） #    同义词  文本分析基础  参数说明 #  同义词分词过滤器可以使用以下参数进行配置：
   参数 必需/可选 数据类型 描述     synonyms 必须指定 synonyms 或 synonyms_path 字符串 在配置中直接定义的同义词规则列表。   synonyms_path 必须指定 synonyms 或 synonyms_path 字符串 包含同义词规则的文件的路径（可以是绝对路径，也可以是相对于配置目录的相对路径）。   lenient 可选 布尔值 加载规则配置时是否忽略异常。默认值为 false。   format 可选 字符串 指定用于确定 Easysearch 如何定义和解释同义词的格式。有效值为：
- solr
- wordnet
默认值为 solr。   expand 可选 布尔值 是否扩展等效的同义词规则。默认值为 true。"
---


# Synonym 分词过滤器

`synonym` 分词过滤器允许将多个术语映射到单个术语，或在单词之间创建等价组，从而提高搜索的灵活性。

## 相关指南（先读这些）

- [同义词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-synonyms.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参数说明

同义词分词过滤器可以使用以下参数进行配置：

| 参数            | 必需/可选                              | 数据类型 | 描述                                                                                                                                                                                                                                                                                                                                                |
| --------------- | -------------------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `synonyms`      | 必须指定 `synonyms` 或 `synonyms_path` | 字符串   | 在配置中直接定义的同义词规则列表。                                                                                                                                                                                                                                                                                                                  |
| `synonyms_path` | 必须指定 `synonyms` 或 `synonyms_path` | 字符串   | 包含同义词规则的文件的路径（可以是绝对路径，也可以是相对于配置目录的相对路径）。                                                                                                                                                                                                                                                                    |
| `lenient`       | 可选                                   | 布尔值   | 加载规则配置时是否忽略异常。默认值为 `false`。                                                                                                                                                                                                                                                                                                      |
| `format`        | 可选                                   | 字符串   | 指定用于确定 Easysearch 如何定义和解释同义词的格式。有效值为：<br> - `solr`<br> - `wordnet`<br> 默认值为 `solr`。                                                                                                                                                                                                                                   |
| `expand`        | 可选                                   | 布尔值   | 是否扩展等效的同义词规则。默认值为 `true`。<br> 例如：<br> 如果同义词定义为 `"quick, fast"` 且 `expand` 设置为 `true`，则同义词规则配置如下：<br> <br>- `quick => quick`<br>- `quick => fast`<br>- `fast => quick`<br>- `fast => fast`<br> <br>如果 `expand` 设置为 `false`，则同义词规则配置如下：<br> <br>- `quick => quick`<br>- `fast => quick` |

## 参考样例： Solr 格式

以下示例请求创建了一个名为 `my-synonym-index` 的新索引，并配置了一个带有同义词过滤器的分词器。该过滤器使用默认的 `Solr` 规则格式进行配置。

```
PUT /my-synonym-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "car, automobile",
            "quick, fast, speedy",
            "laptop => computer"
          ]
        }
      },
      "analyzer": {
        "my_synonym_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
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
GET /my-synonym-index/_analyze
{
  "analyzer": "my_synonym_analyzer",
  "text": "The quick dog jumps into the car with a laptop"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "the",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "quick",
      "start_offset": 4,
      "end_offset": 9,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "fast",
      "start_offset": 4,
      "end_offset": 9,
      "type": "SYNONYM",
      "position": 1
    },
    {
      "token": "speedy",
      "start_offset": 4,
      "end_offset": 9,
      "type": "SYNONYM",
      "position": 1
    },
    {
      "token": "dog",
      "start_offset": 10,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "jumps",
      "start_offset": 14,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "into",
      "start_offset": 20,
      "end_offset": 24,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "the",
      "start_offset": 25,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "car",
      "start_offset": 29,
      "end_offset": 32,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "automobile",
      "start_offset": 29,
      "end_offset": 32,
      "type": "SYNONYM",
      "position": 6
    },
    {
      "token": "with",
      "start_offset": 33,
      "end_offset": 37,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "a",
      "start_offset": 38,
      "end_offset": 39,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "computer",
      "start_offset": 40,
      "end_offset": 46,
      "type": "SYNONYM",
      "position": 9
    }
  ]
}
```

## 参考样例：WordNet 格式

以下示例请求创建了一个名为 `my-wordnet-index` 的新索引，并配置了一个带有同义词过滤器的分词器。该过滤器是按照 `WordNet` 规则格式配置的。

```
PUT /my-wordnet-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_wordnet_synonym_filter": {
          "type": "synonym",
          "format": "wordnet",
          "synonyms": [
            "s(100000001,1,'fast',v,1,0).",
            "s(100000001,2,'quick',v,1,0).",
            "s(100000001,3,'swift',v,1,0)."
          ]
        }
      },
      "analyzer": {
        "my_wordnet_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_wordnet_synonym_filter"
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
GET /my-wordnet-index/_analyze
{
  "analyzer": "my_wordnet_analyzer",
  "text": "I have a fast car"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "i",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "have",
      "start_offset": 2,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "a",
      "start_offset": 7,
      "end_offset": 8,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "fast",
      "start_offset": 9,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "quick",
      "start_offset": 9,
      "end_offset": 13,
      "type": "SYNONYM",
      "position": 3
    },
    {
      "token": "swift",
      "start_offset": 9,
      "end_offset": 13,
      "type": "SYNONYM",
      "position": 3
    },
    {
      "token": "car",
      "start_offset": 14,
      "end_offset": 17,
      "type": "<ALPHANUM>",
      "position": 4
    }
  ]
}
```

