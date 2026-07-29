---
title: "搜索分析器参数（Search Analyzer）"
date: 0001-01-01
summary: "Search Analyzer 参数 #  search_analyzer 参数指定查询时使用的分析器，可以与索引时的 analyzer 不同。这允许在写入和查询时使用不同的文本分析策略。
相关指南（先读这些） #    Analyzer 参数  文本分析  映射基础  分析器选择优先级 #  Easysearch 按以下优先级选择查询时分析器：
 查询中指定的 analyzer 参数（如 match 查询的 analyzer 字段） 映射中字段的 search_analyzer 索引设置 analysis.analyzer.default_search 映射中字段的 analyzer standard 分析器  示例 #  写入时最大化召回，查询时精确匹配 #  PUT my-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;ik_smart_search&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;ik_smart&#34; } } } }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;ik_max_word&#34;, &#34;search_analyzer&#34;: &#34;ik_smart&#34; } } } }    阶段 分析器 策略     索引时 ik_max_word 最细粒度分词，生成更多词项，最大化召回   查询时 ik_smart 智能分词，生成更少词项，提高精确度    同义词扩展只在查询时生效 #  &#34;title&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;standard&#34;, &#34;search_analyzer&#34;: &#34;synonym_analyzer&#34; } 这样索引时不扩展同义词（节省空间），查询时通过同义词分析器扩展查询词。"
---


# Search Analyzer 参数

`search_analyzer` 参数指定查询时使用的分析器，可以与索引时的 `analyzer` 不同。这允许在写入和查询时使用不同的文本分析策略。

## 相关指南（先读这些）

- [Analyzer 参数]({{< relref "analyzer" >}})
- [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})

## 分析器选择优先级

Easysearch 按以下优先级选择查询时分析器：

1. 查询中指定的 `analyzer` 参数（如 `match` 查询的 `analyzer` 字段）
2. 映射中字段的 `search_analyzer`
3. 索引设置 `analysis.analyzer.default_search`
4. 映射中字段的 `analyzer`
5. `standard` 分析器

## 示例

### 写入时最大化召回，查询时精确匹配

```json
PUT my-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "ik_smart_search": {
          "type": "custom",
          "tokenizer": "ik_smart"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

| 阶段 | 分析器 | 策略 |
|------|--------|------|
| 索引时 | `ik_max_word` | 最细粒度分词，生成更多词项，最大化召回 |
| 查询时 | `ik_smart` | 智能分词，生成更少词项，提高精确度 |

### 同义词扩展只在查询时生效

```json
"title": {
  "type": "text",
  "analyzer": "standard",
  "search_analyzer": "synonym_analyzer"
}
```

这样索引时不扩展同义词（节省空间），查询时通过同义词分析器扩展查询词。

## 常见用法

| 场景 | 索引分析器 | 查询分析器 |
|------|-----------|-----------|
| 中文搜索 | `ik_max_word`（最大召回） | `ik_smart`（精确匹配） |
| 同义词扩展 | `standard` | 带同义词过滤的自定义分析器 |
| 拼音搜索 | `pinyin` | `standard`（用户输入可能是拼音也可能是汉字） |

## 注意事项

- 如果不设置 `search_analyzer`，查询时默认使用字段的 `analyzer`
- 索引时和查询时使用不同分析器时，需确保两者产生的词项能够正确匹配
- 可以在具体查询中通过 `analyzer` 参数临时覆盖

