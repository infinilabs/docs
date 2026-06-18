---
title: "短语索引（Index Phrases）"
date: 0001-01-01
summary: "短语索引（Index Phrases） #  启用短语查询的优化。当启用此参数时，Easysearch 会为短语查询构建额外的索引结构。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;index_phrases&#34;: true } } } } 适用的字段类型 #   text match_only_text  默认值 #   false  说明 #  启用 index_phrases 后，短语查询的性能会显著提高，但会增加索引大小。
相关参数 #    前缀索引（Index Prefixes）  索引参数（Index）  "
---


# 短语索引（Index Phrases）

启用短语查询的优化。当启用此参数时，Easysearch 会为短语查询构建额外的索引结构。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "index_phrases": true
      }
    }
  }
}
```

## 适用的字段类型

- `text`
- `match_only_text`

## 默认值

- `false`

## 说明

启用 `index_phrases` 后，短语查询的性能会显著提高，但会增加索引大小。

## 相关参数

- [前缀索引（Index Prefixes）]({{< relref "index_prefixes.md" >}})
- [索引参数（Index）]({{< relref "index.md" >}})

