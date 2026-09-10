---
title: "搜索分析器（Search Analyzer）"
date: 0001-01-01
summary: "搜索分词器 #  搜索分词器是在查询时指定的，当你对text字段进行全文匹配查询时，它会被用于分析被查询字符串。
搜索分词器的生效流程 #  在查询时确定对查询字符串使用哪种分词器，Easysearch 会按以下顺序检查以下参数：
 查询的 analyzer 参数 字段的 search_analyzer 映射参数 索引设置 analysis.analyzer.default_search 字段的 analyzer 映射参数 standard 分词器（默认分词器）   在大多数情况下，没有必要指定与索引分词器不同的搜索分词器，而且这样做可能会对搜索结果的相关性产生负面影响，或者导致出现意想不到的搜索结果。
 为查询内容指定搜索分词器 #  在查询时，你可以在 analyzer 字段中指定想要使用的分词器：
GET shakespeare/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;text_entry&#34;: { &#34;query&#34;: &#34;speak the truth&#34;, &#34;analyzer&#34;: &#34;english&#34; } } } } 为字段指定搜索分词器 #  在创建索引映射时，你可以为每个文本字段提供 search_analyzer 参数。在提供 search_analyzer 时，你还必须提供 analyzer 参数，该参数指定在索引创建时的写入索引分词器内应用。
例如，以下请求将 simple 分词器指定为 text_entry 字段的索引分词器，将 whitespace 分词器指定为该字段的搜索分词器：
PUT testindex { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;text_entry&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;simple&#34;, &#34;search_analyzer&#34;: &#34;whitespace&#34; } } } } 为索引指定默认的搜索分词器 #  如果你希望在搜索时使用同一个分词器来解析所有的查询，你可以在 analysis."
---


# 搜索分词器

搜索分词器是在查询时指定的，当你对`text`字段进行全文匹配查询时，它会被用于分析被查询字符串。

## 搜索分词器的生效流程

在查询时确定对查询字符串使用哪种分词器，Easysearch 会按以下顺序检查以下参数：

1. 查询的 `analyzer` 参数
2. 字段的 `search_analyzer` 映射参数
3. 索引设置 `analysis.analyzer.default_search`
4. 字段的 `analyzer` 映射参数
5. `standard` 分词器（默认分词器）

> 在大多数情况下，没有必要指定与索引分词器不同的搜索分词器，而且这样做可能会对搜索结果的相关性产生负面影响，或者导致出现意想不到的搜索结果。

## 为查询内容指定搜索分词器

在查询时，你可以在 `analyzer` 字段中指定想要使用的分词器：

```
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": {
        "query": "speak the truth",
        "analyzer": "english"
      }
    }
  }
}
```

## 为字段指定搜索分词器

在创建索引映射时，你可以为每个文本字段提供 `search_analyzer` 参数。在提供 `search_analyzer` 时，你还必须提供 `analyzer` 参数，该参数指定在索引创建时的写入索引分词器内应用。

例如，以下请求将 `simple` 分词器指定为 `text_entry` 字段的索引分词器，将 `whitespace` 分词器指定为该字段的搜索分词器：

```
PUT testindex
{
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "text",
        "analyzer": "simple",
        "search_analyzer": "whitespace"
      }
    }
  }
}
```

## 为索引指定默认的搜索分词器

如果你希望在搜索时使用同一个分词器来解析所有的查询，你可以在 `analysis.analyzer.default_search` 设置中指定该搜索分词器。在提供 `analysis.analyzer.default_search` 时，你还必须提供 `analysis.analyzer.default` 参数，该参数指定了在创建索引时要使用的索引分词器。

例如，以下请求将 `simple` 分词器指定为 `testindex` 索引的索引分词器，并将 `whitespace` 分词器指定为该索引的搜索分词器

```
PUT testindex
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "simple"
        },
        "default_search": {
          "type": "whitespace"
        }
      }
    }
  }
}
```

