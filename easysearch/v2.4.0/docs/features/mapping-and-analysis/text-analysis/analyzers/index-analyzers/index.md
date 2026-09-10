---
title: "索引分析器（Index Analyzer）"
date: 0001-01-01
summary: "索引分词器 #  索引分词器是在索引创建时指定的，在文档写入时对文本字段进行分词。
写入索引分词器的生效流程 #  为了确定在对文档建立索引时对某个字段使用哪种分词器，Easysearch 会按以下顺序检查以下参数：
 字段的分词器映射参数，即analyzer analysis.analyzer.default 索引设置 standard标准分词器（默认分词器）   在指定索引分词器时，请记住，在大多数情况下，为索引中的每个文本字段指定一个分词器效果最佳。使用相同的分词器在文本字段（在建立索引时）和查询字符串（在查询时），可确保搜索所使用的词项与存储在索引中的词项相同。
 为字段指定索引分词器 #  在创建索引映射时，可以为每个文本字段提供 analyzer（分词器）参数。例如，以下请求为 text_entry 字段指定了 simple 简单分词器：
PUT testindex { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;text_entry&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;simple&#34; } } } } 为索引指定默认索引分词器 #  如果想对索引中的所有文本字段使用相同的分词器，则可以在索引设置中按如下方式进行指定analysis.analyzer.default：
PUT testindex { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;default&#34;: { &#34;type&#34;: &#34;simple&#34; } } } } }  如果您未指定默认分词器，那么将使用standard标准分词器。
 "
---


# 索引分词器

索引分词器是在索引创建时指定的，在文档写入时对文本字段进行分词。

## 写入索引分词器的生效流程

为了确定在对文档建立索引时对某个字段使用哪种分词器，Easysearch 会按以下顺序检查以下参数：

- 字段的分词器映射参数，即`analyzer`
- `analysis.analyzer.default` 索引设置
- `standard`标准分词器（默认分词器）

> 在指定索引分词器时，请记住，在大多数情况下，为索引中的每个文本字段指定一个分词器效果最佳。使用相同的分词器在文本字段（在建立索引时）和查询字符串（在查询时），可确保搜索所使用的词项与存储在索引中的词项相同。

## 为字段指定索引分词器

在创建索引映射时，可以为每个文本字段提供 `analyzer`（分词器）参数。例如，以下请求为 `text_entry` 字段指定了 `simple` 简单分词器：

```
PUT testindex
{
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "text",
        "analyzer": "simple"
      }
    }
  }
}
```

## 为索引指定默认索引分词器

如果想对索引中的所有文本字段使用相同的分词器，则可以在索引设置中按如下方式进行指定`analysis.analyzer.default`：

```
PUT testindex
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "simple"
        }
      }
    }
  }
}
```

> 如果您未指定默认分词器，那么将使用`standard`标准分词器。

