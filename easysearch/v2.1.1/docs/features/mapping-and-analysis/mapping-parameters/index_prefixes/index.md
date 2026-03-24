---
title: "前缀索引（Index Prefixes）"
date: 0001-01-01
summary: "前缀索引（Index Prefixes） #  启用前缀查询的优化。当启用此参数时，Easysearch 会为前缀查询构建额外的索引结构。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;username&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;index_prefixes&#34;: {} } } } } 适用的字段类型 #   text match_only_text  参数选项 #  { &#34;min_chars&#34;: 0, &#34;max_chars&#34;: 20 }  min_chars: 最小前缀长度（默认值：0） max_chars: 最大前缀长度（默认值：20）  说明 #  启用 index_prefixes 后，通配符查询和前缀查询的性能会显著提高。但会增加索引大小。较小的 min_chars 和 max_chars 值会使索引更大。
相关参数 #    短语索引（Index Phrases）  索引参数（Index）  "
---


# 前缀索引（Index Prefixes）

启用前缀查询的优化。当启用此参数时，Easysearch 会为前缀查询构建额外的索引结构。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "username": {
        "type": "text",
        "index_prefixes": {}
      }
    }
  }
}
```

## 适用的字段类型

- `text`
- `match_only_text`

## 参数选项

```json
{
  "min_chars": 0,
  "max_chars": 20
}
```

- **min_chars**: 最小前缀长度（默认值：0）
- **max_chars**: 最大前缀长度（默认值：20）

## 说明

启用 `index_prefixes` 后，通配符查询和前缀查询的性能会显著提高。但会增加索引大小。较小的 `min_chars` 和 `max_chars` 值会使索引更大。

## 相关参数

- [短语索引（Index Phrases）]({{< relref "index_phrases.md" >}})
- [索引参数（Index）]({{< relref "index.md" >}})

