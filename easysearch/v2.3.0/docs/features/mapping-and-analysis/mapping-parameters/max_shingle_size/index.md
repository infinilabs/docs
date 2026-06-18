---
title: "最大短语大小（Max Shingle Size）"
date: 0001-01-01
summary: "最大短语大小（Max Shingle Size） #  搜索自动完成时使用的最大 shingle 大小。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;text&#34;: { &#34;type&#34;: &#34;search_as_you_type&#34;, &#34;max_shingle_size&#34;: 3 } } } } 适用的字段类型 #   search_as_you_type（mapper-extras 模块）  默认值 #   3  说明 #  search_as_you_type 字段类型生成额外的子字段以支持即时搜索。max_shingle_size 参数控制用于前缀查询的 shingle 的最大大小。值越大，索引越大，但查询越快。
相关参数 #    分析器参数（Analyzer）  搜索分析器参数（Search Analyzer）  "
---


# 最大短语大小（Max Shingle Size）

搜索自动完成时使用的最大 shingle 大小。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "text": {
        "type": "search_as_you_type",
        "max_shingle_size": 3
      }
    }
  }
}
```

## 适用的字段类型

- `search_as_you_type`（mapper-extras 模块）

## 默认值

- `3`

## 说明

`search_as_you_type` 字段类型生成额外的子字段以支持即时搜索。`max_shingle_size` 参数控制用于前缀查询的 shingle 的最大大小。值越大，索引越大，但查询越快。

## 相关参数

- [分析器参数（Analyzer）]({{< relref "analyzer.md" >}})
- [搜索分析器参数（Search Analyzer）]({{< relref "search_analyzer.md" >}})

