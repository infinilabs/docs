---
title: "短语搜索分析器（Search Quote Analyzer）"
date: 0001-01-01
summary: "短语搜索分析器（Search Quote Analyzer） #  指定在处理带引号的短语查询时使用的分析器。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;standard&#34;, &#34;search_quote_analyzer&#34;: &#34;keyword&#34; } } } } 适用的字段类型 #   text  默认值 #   如果未指定，使用 search_analyzer 的值 如果 search_analyzer 也未指定，使用 analyzer 的值  说明 #  在执行带引号短语查询时，可以使用不同的分析器来改进相关性。例如，可以禁用停用词移除以获得更精确的短语匹配。
相关参数 #    分析器参数（Analyzer）  搜索分析器参数（Search Analyzer）  "
---


# 短语搜索分析器（Search Quote Analyzer）

指定在处理带引号的短语查询时使用的分析器。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",
        "search_quote_analyzer": "keyword"
      }
    }
  }
}
```

## 适用的字段类型

- `text`

## 默认值

- 如果未指定，使用 `search_analyzer` 的值
- 如果 `search_analyzer` 也未指定，使用 `analyzer` 的值

## 说明

在执行带引号短语查询时，可以使用不同的分析器来改进相关性。例如，可以禁用停用词移除以获得更精确的短语匹配。

## 相关参数

- [分析器参数（Analyzer）]({{< relref "analyzer.md" >}})
- [搜索分析器参数（Search Analyzer）]({{< relref "search_analyzer.md" >}})

