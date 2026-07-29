---
title: "关键字标记分词过滤器（Keyword Marker）"
date: 0001-01-01
summary: "Keyword Marker 分词过滤器 #  keyword_marker 分词过滤器用于防止某些词元被词干提取器或其他过滤器修改。该过滤器通过将指定的词元标记为关键词来实现这一点，这样就能避免词干提取或其他方式的处理。这确保了特定的单词能保持其原始形式。
相关指南（先读这些） #    文本分析：词干提取  文本分析：规范化  参数说明 #  关键词标记分词过滤器可以使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     ignore_case 可选 布尔值 在匹配关键词时是否忽略字母大小写。默认值为false。   keywords 若未设置 keywords_path 或 keywords_pattern 则为必需 字符串列表 要标记为关键词的词元列表。   keywords_path 若未设置 keywords 或 keywords_pattern 则为必需 字符串 关键词列表的路径（相对于配置目录或绝对路径）。   keywords_pattern 若未设置 keywords 或 keywords_path 则为必需 字符串 用于匹配要标记为关键词的词元的正则表达式。    参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个带有关键词标记过滤器的分词器。该过滤器将单词 example 标记为关键词："
---


# Keyword Marker 分词过滤器

`keyword_marker` 分词过滤器用于防止某些词元被词干提取器或其他过滤器修改。该过滤器通过将指定的词元标记为关键词来实现这一点，这样就能避免词干提取或其他方式的处理。这确保了特定的单词能保持其原始形式。

## 相关指南（先读这些）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

关键词标记分词过滤器可以使用以下参数进行配置。

| 参数               | 必需/可选                                               | 数据类型   | 描述                                                |
| ------------------ | ------------------------------------------------------- | ---------- | --------------------------------------------------- |
| `ignore_case`      | 可选                                                    | 布尔值     | 在匹配关键词时是否忽略字母大小写。默认值为`false`。 |
| `keywords`         | 若未设置 `keywords_path` 或 `keywords_pattern` 则为必需 | 字符串列表 | 要标记为关键词的词元列表。                          |
| `keywords_path`    | 若未设置 `keywords` 或 `keywords_pattern` 则为必需      | 字符串     | 关键词列表的路径（相对于配置目录或绝对路径）。      |
| `keywords_pattern` | 若未设置 `keywords` 或 `keywords_path` 则为必需         | 字符串     | 用于匹配要标记为关键词的词元的正则表达式。          |

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个带有关键词标记过滤器的分词器。该过滤器将单词 `example` 标记为关键词：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "keyword_marker_filter", "stemmer"]
        }
      },
      "filter": {
        "keyword_marker_filter": {
          "type": "keyword_marker",
          "keywords": ["example"]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
GET /my_index/_analyze
{
  "analyzer": "custom_analyzer",
  "text": "Favorite example"
}
```

返回中包含了生成的词元。请注意，虽然单词 `favorite` 进行了词干提取，但单词 `example` 未进行词干提取，因为它被标记为了关键词。

```
{
  "tokens": [
    {
      "token": "favorit",
      "start_offset": 0,
      "end_offset": 8,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "example",
      "start_offset": 9,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

你可以通过在 `_analyze` 查询中添加以下参数，进一步研究关键词标记分词过滤器的影响：

```
GET /my_index/_analyze
{
  "analyzer": "custom_analyzer",
  "text": "This is an Easysearch example demonstrating keyword marker.",
  "explain": true,
  "attributes": "keyword"
}
```

这将在返回内容中生成类似如下的更多详细信息：

```
{
    "name": "porter_stem",
    "tokens": [
      ...
      {
        "token": "example",
        "start_offset": 22,
        "end_offset": 29,
        "type": "<ALPHANUM>",
        "position": 4,
        "keyword": true
      },
      {
        "token": "demonstr",
        "start_offset": 30,
        "end_offset": 43,
        "type": "<ALPHANUM>",
        "position": 5,
        "keyword": false
      },
      ...
    ]
}
```

