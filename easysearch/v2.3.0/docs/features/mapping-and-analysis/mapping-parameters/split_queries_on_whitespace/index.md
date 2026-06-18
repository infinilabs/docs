---
title: "空白符拆分查询（Split Queries on Whitespace）"
date: 0001-01-01
summary: "空白符拆分查询（Split Queries on Whitespace） #  确定查询是否在空白处拆分。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;status&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;split_queries_on_whitespace&#34;: true } } } } 适用的字段类型 #   keyword wildcard  默认值 #   false  说明 #  当设置为 true 时，查询字符串在空白处拆分成多个术语。这在索引短语（如空格分隔的标签）的关键字字段中很有用。
相关参数 #    分析器参数（Analyzer）  "
---


# 空白符拆分查询（Split Queries on Whitespace）

确定查询是否在空白处拆分。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "status": {
        "type": "keyword",
        "split_queries_on_whitespace": true
      }
    }
  }
}
```

## 适用的字段类型

- `keyword`
- `wildcard`

## 默认值

- `false`

## 说明

当设置为 `true` 时，查询字符串在空白处拆分成多个术语。这在索引短语（如空格分隔的标签）的关键字字段中很有用。

## 相关参数

- [分析器参数（Analyzer）]({{< relref "analyzer.md" >}})

