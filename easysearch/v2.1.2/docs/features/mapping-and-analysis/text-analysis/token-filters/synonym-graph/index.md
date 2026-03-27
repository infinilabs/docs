---
title: "同义词图分词过滤器（Synonym Graph）"
date: 0001-01-01
summary: "Synonym Graph 分词过滤器 #  synonym_graph 分词过滤器是同义词分词过滤器的更高级版本。它支持多词同义词，并能跨多个词元处理同义词，这使得它非常适合处理短语或者那些词元之间关系很重要的场景。
相关指南（先读这些） #    文本分析：同义词  文本分析：规范化  参数说明 #  同义词图分词过滤器可以使用以下参数进行配置：
   参数 必需/可选 数据类型 描述     synonyms 必须指定 synonyms 或 synonyms_path 字符串 在配置中直接定义的同义词规则列表。   synonyms_path 必须指定 synonyms 或 synonyms_path 字符串 包含同义词规则的文件的路径（可以是绝对路径，也可以是相对于配置目录的相对路径）。   lenient 可选 布尔值 在加载规则配置时是否忽略异常。默认值为 false。   format 可选 字符串 指定用于确定 Easysearch 如何定义和解释同义词的格式。有效值为：
- solr
- wordnet
默认值为 solr。   expand 可选 布尔值 是否扩展等效的同义词规则。默认值为 true。"
---


# Synonym Graph 分词过滤器

`synonym_graph` 分词过滤器是同义词分词过滤器的更高级版本。它支持多词同义词，并能跨多个词元处理同义词，这使得它非常适合处理短语或者那些词元之间关系很重要的场景。

## 相关指南（先读这些）

- [文本分析：同义词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-synonyms.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

同义词图分词过滤器可以使用以下参数进行配置：

| 参数            | 必需/可选                              | 数据类型 | 描述                                                                                                                                                                                                                                                                                                                                                                            |
| --------------- | -------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `synonyms`      | 必须指定 `synonyms` 或 `synonyms_path` | 字符串   | 在配置中直接定义的同义词规则列表。                                                                                                                                                                                                                                                                                                                                              |
| `synonyms_path` | 必须指定 `synonyms` 或 `synonyms_path` | 字符串   | 包含同义词规则的文件的路径（可以是绝对路径，也可以是相对于配置目录的相对路径）。                                                                                                                                                                                                                                                                                                |
| `lenient`       | 可选                                   | 布尔值   | 在加载规则配置时是否忽略异常。默认值为 `false`。                                                                                                                                                                                                                                                                                                                                |
| `format`        | 可选                                   | 字符串   | 指定用于确定 Easysearch 如何定义和解释同义词的格式。有效值为：<br> - `solr`<br> - `wordnet`<br> 默认值为 `solr`。                                                                                                                                                                                                                                                               |
| `expand`        | 可选                                   | 布尔值   | 是否扩展等效的同义词规则。默认值为 `true`。<br>例如：<br>若同义词定义为 “quick, fast” 且 `expand` 设置为 `true`，则同义词规则配置如下：<br> <br>- `quick` 映射为 `quick`<br>- `quick` 映射为 `fast`<br>- `fast` 映射为 `quick`<br>- `fast` 映射为 `fast`<br> <br>若 `expand` 设置为 `false`，则同义词规则配置如下：<br> <br>- `quick` 映射为 `quick`<br>- `fast` 映射为 `quick` |

## 参考样例： Solr 格式

以下示例请求创建了一个名为 `my-index` 的新索引，并配置了一个带有同义词图过滤器的分词器。该过滤器采用默认的 `Solr` 规则格式进行配置。

```
PUT /my-car-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_graph_filter": {
          "type": "synonym_graph",
          "synonyms": [
            "sports car, race car",
            "fast car, speedy vehicle",
            "luxury car, premium vehicle",
            "electric car, EV"
          ]
        }
      },
      "analyzer": {
        "my_synonym_graph_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_graph_filter"
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
GET /my-car-index/_analyze
{
  "analyzer": "my_synonym_graph_analyzer",
  "text": "I just bought a sports car and it is a fast car."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {"token": "i","start_offset": 0,"end_offset": 1,"type": "<ALPHANUM>","position": 0},
    {"token": "just","start_offset": 2,"end_offset": 6,"type": "<ALPHANUM>","position": 1},
    {"token": "bought","start_offset": 7,"end_offset": 13,"type": "<ALPHANUM>","position": 2},
    {"token": "a","start_offset": 14,"end_offset": 15,"type": "<ALPHANUM>","position": 3},
    {"token": "race","start_offset": 16,"end_offset": 26,"type": "SYNONYM","position": 4},
    {"token": "sports","start_offset": 16,"end_offset": 22,"type": "<ALPHANUM>","position": 4,"positionLength": 2},
    {"token": "car","start_offset": 16,"end_offset": 26,"type": "SYNONYM","position": 5,"positionLength": 2},
    {"token": "car","start_offset": 23,"end_offset": 26,"type": "<ALPHANUM>","position": 6},
    {"token": "and","start_offset": 27,"end_offset": 30,"type": "<ALPHANUM>","position": 7},
    {"token": "it","start_offset": 31,"end_offset": 33,"type": "<ALPHANUM>","position": 8},
    {"token": "is","start_offset": 34,"end_offset": 36,"type": "<ALPHANUM>","position": 9},
    {"token": "a","start_offset": 37,"end_offset": 38,"type": "<ALPHANUM>","position": 10},
    {"token": "speedy","start_offset": 39,"end_offset": 47,"type": "SYNONYM","position": 11},
    {"token": "fast","start_offset": 39,"end_offset": 43,"type": "<ALPHANUM>","position": 11,"positionLength": 2},
    {"token": "vehicle","start_offset": 39,"end_offset": 47,"type": "SYNONYM","position": 12,"positionLength": 2},
    {"token": "car","start_offset": 44,"end_offset": 47,"type": "<ALPHANUM>","position": 13}
  ]
}
```

## 参考样例： WordNet 格式

以下示例请求创建了一个名为 `my-wordnet-index` 的新索引，并配置了一个带有同义词图过滤器的分词器。该过滤器采用 WordNet 规则格式进行配置。

```
PUT /my-wordnet-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_graph_filter": {
          "type": "synonym_graph",
          "format": "wordnet",
          "synonyms": [
            "s(100000001, 1, 'sports car', n, 1, 0).",
            "s(100000001, 2, 'race car', n, 1, 0).",
            "s(100000001, 3, 'fast car', n, 1, 0).",
            "s(100000001, 4, 'speedy vehicle', n, 1, 0)."
          ]
        }
      },
      "analyzer": {
        "my_synonym_graph_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_graph_filter"
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
  "analyzer": "my_synonym_graph_analyzer",
  "text": "I just bought a sports car and it is a fast car."
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {"token": "i","start_offset": 0,"end_offset": 1,"type": "<ALPHANUM>","position": 0},
    {"token": "just","start_offset": 2,"end_offset": 6,"type": "<ALPHANUM>","position": 1},
    {"token": "bought","start_offset": 7,"end_offset": 13,"type": "<ALPHANUM>","position": 2},
    {"token": "a","start_offset": 14,"end_offset": 15,"type": "<ALPHANUM>","position": 3},
    {"token": "race","start_offset": 16,"end_offset": 26,"type": "SYNONYM","position": 4},
    {"token": "fast","start_offset": 16,"end_offset": 26,"type": "SYNONYM","position": 4,"positionLength": 2},
    {"token": "speedy","start_offset": 16,"end_offset": 26,"type": "SYNONYM","position": 4,"positionLength": 3},
    {"token": "sports","start_offset": 16,"end_offset": 22,"type": "<ALPHANUM>","position": 4,"positionLength": 4},
    {"token": "car","start_offset": 16,"end_offset": 26,"type": "SYNONYM","position": 5,"positionLength": 4},
    {"token": "car","start_offset": 16,"end_offset": 26,"type": "SYNONYM","position": 6,"positionLength": 3},
    {"token": "vehicle","start_offset": 16,"end_offset": 26,"type": "SYNONYM","position": 7,"positionLength": 2},
    {"token": "car","start_offset": 23,"end_offset": 26,"type": "<ALPHANUM>","position": 8},
    {"token": "and","start_offset": 27,"end_offset": 30,"type": "<ALPHANUM>","position": 9},
    {"token": "it","start_offset": 31,"end_offset": 33,"type": "<ALPHANUM>","position": 10},
    {"token": "is","start_offset": 34,"end_offset": 36,"type": "<ALPHANUM>","position": 11},
    {"token": "a","start_offset": 37,"end_offset": 38,"type": "<ALPHANUM>","position": 12},
    {"token": "sports","start_offset": 39,"end_offset": 47,"type": "SYNONYM","position": 13},
    {"token": "race","start_offset": 39,"end_offset": 47,"type": "SYNONYM","position": 13,"positionLength": 2},
    {"token": "speedy","start_offset": 39,"end_offset": 47,"type": "SYNONYM","position": 13,"positionLength": 3},
    {"token": "fast","start_offset": 39,"end_offset": 43,"type": "<ALPHANUM>","position": 13,"positionLength": 4},
    {"token": "car","start_offset": 39,"end_offset": 47,"type": "SYNONYM","position": 14,"positionLength": 4},
    {"token": "car","start_offset": 39,"end_offset": 47,"type": "SYNONYM","position": 15,"positionLength": 3},
    {"token": "vehicle","start_offset": 39,"end_offset": 47,"type": "SYNONYM","position": 16,"positionLength": 2},
    {"token": "car","start_offset": 44,"end_offset": 47,"type": "<ALPHANUM>","position": 17}
  ]
}
```

