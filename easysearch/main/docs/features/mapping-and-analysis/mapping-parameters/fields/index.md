---
title: "多字段参数（Fields）"
date: 0001-01-01
summary: "Fields 参数（Multi-fields） #  fields 参数允许你对同一个字段以多种方式进行索引。这是 Easysearch 中最常用的映射技巧之一，典型场景是：一个字段同时需要全文搜索（text）和精确匹配/排序/聚合（keyword）。
相关指南（先读这些） #    映射基础  映射模式与最佳实践  基本语法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;fields&#34;: { &#34;raw&#34;: { &#34;type&#34;: &#34;keyword&#34; } } } } } } 上面的映射中，title 字段被索引为 text（用于全文搜索），同时通过 fields.raw 创建了一个 keyword 子字段（用于精确匹配、排序和聚合）。
查询时使用方式：
 全文搜索：&quot;match&quot;: { &quot;title&quot;: &quot;搜索引擎&quot; } 精确匹配：&quot;term&quot;: { &quot;title.raw&quot;: &quot;Easysearch 分布式搜索引擎&quot; } 排序：&quot;sort&quot;: [{ &quot;title.raw&quot;: &quot;asc&quot; }] 聚合：&quot;aggs&quot;: { &quot;titles&quot;: { &quot;terms&quot;: { &quot;field&quot;: &quot;title."
---


# Fields 参数（Multi-fields）

`fields` 参数允许你对同一个字段以多种方式进行索引。这是 Easysearch 中最常用的映射技巧之一，典型场景是：一个字段同时需要全文搜索（`text`）和精确匹配/排序/聚合（`keyword`）。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式与最佳实践]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

## 基本语法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "raw": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

上面的映射中，`title` 字段被索引为 `text`（用于全文搜索），同时通过 `fields.raw` 创建了一个 `keyword` 子字段（用于精确匹配、排序和聚合）。

查询时使用方式：

- 全文搜索：`"match": { "title": "搜索引擎" }`
- 精确匹配：`"term": { "title.raw": "Easysearch 分布式搜索引擎" }`
- 排序：`"sort": [{ "title.raw": "asc" }]`
- 聚合：`"aggs": { "titles": { "terms": { "field": "title.raw" } } }`

## 常见用法

### 1. text + keyword（最典型）

```json
"name": {
  "type": "text",
  "analyzer": "ik_max_word",
  "fields": {
    "keyword": {
      "type": "keyword",
      "ignore_above": 256
    }
  }
}
```

> **提示**：Easysearch 的动态映射默认就为字符串字段创建这种 text + keyword 的多字段结构。

### 2. 同一字段使用不同分析器

```json
"content": {
  "type": "text",
  "analyzer": "ik_max_word",
  "fields": {
    "standard": {
      "type": "text",
      "analyzer": "standard"
    },
    "pinyin": {
      "type": "text",
      "analyzer": "pinyin"
    }
  }
}
```

这样可以在搜索时选择最合适的分析器：

- `content`：中文分词搜索
- `content.standard`：英文/通用搜索
- `content.pinyin`：拼音搜索

### 3. keyword + 不同 normalizer

```json
"status": {
  "type": "keyword",
  "fields": {
    "lower": {
      "type": "keyword",
      "normalizer": "lowercase"
    }
  }
}
```

## 注意事项

| 要点 | 说明 |
|------|------|
| 子字段继承 | 子字段**不会**继承父字段的参数，必须独立配置 |
| 存储开销 | 每个子字段都会独立存储索引数据，增加磁盘空间 |
| 动态映射 | 默认动态映射会为字符串自动创建 `text` + `keyword` 子字段 |
| 更新限制 | 已有索引的 `fields` 可以追加新子字段，但不能修改已有子字段 |
| 查询方式 | 使用 `字段名.子字段名` 的点号语法访问子字段 |

## 与 copy_to 的对比

| 特性 | fields（多字段） | copy_to |
|------|------------------|---------|
| 作用 | 同一字段多种索引方式 | 将值复制到另一个独立字段 |
| 访问方式 | `field.subfield` | 直接使用目标字段名 |
| _source | 不出现在 `_source` 中 | 不出现在 `_source` 中 |
| 典型场景 | text + keyword | 多个字段合并到一个搜索字段 |

