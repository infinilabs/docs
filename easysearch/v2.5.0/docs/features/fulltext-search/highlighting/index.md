---
title: "高亮"
date: 0001-01-01
description: "高亮的工作方式、三种高亮器类型、参数、性能影响与常见问题排查。"
summary: "高亮 #  高亮用于在返回结果中标记命中的文本片段，提升可读性。Easysearch 在搜索阶段记录哪些文本片段匹配了查询，在返回阶段根据这些信息对原文做截取与包裹（例如 &lt;em&gt;...&lt;/em&gt; 标签）。
基本用法 #  GET /articles/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;content&#34;: &#34;搜索 引擎&#34; } }, &#34;highlight&#34;: { &#34;fields&#34;: { &#34;content&#34;: {} } } } 返回结果中每条命中文档会包含 highlight 字段：
{ &#34;hits&#34;: { &#34;hits&#34;: [ { &#34;_source&#34;: { &#34;content&#34;: &#34;Easysearch 是一款高性能的搜索引擎...&#34; }, &#34;highlight&#34;: { &#34;content&#34;: [ &#34;Easysearch 是一款高性能的&lt;em&gt;搜索&lt;/em&gt;&lt;em&gt;引擎&lt;/em&gt;...&#34; ] } } ] } } 三种高亮器类型 #  Easysearch 支持三种高亮器，通过 type 参数指定：
   类型 说明 优缺点     unified（默认） 基于 Lucene Unified Highlighter，使用 BM25 对片段评分 推荐首选。支持所有字段类型，性能与准确性平衡最佳   plain 基于标准 Lucene Highlighter，逐词高亮 小文本场景尚可，大文本或复杂查询时性能较差   fvh Fast Vector Highlighter，需要 term_vector 设置为 with_positions_offsets 大文本高亮最快，但需要额外索引存储    使用 FVH 高亮器 #  PUT /articles { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;term_vector&#34;: &#34;with_positions_offsets&#34; } } } } GET /articles/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;content&#34;: &#34;搜索 引擎&#34; } }, &#34;highlight&#34;: { &#34;type&#34;: &#34;fvh&#34;, &#34;fields&#34;: { &#34;content&#34;: {} } } } 常用参数 #  GET /articles/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;content&#34;: &#34;搜索 引擎&#34; } }, &#34;highlight&#34;: { &#34;pre_tags&#34;: [&#34;&lt;strong&gt;&#34;], &#34;post_tags&#34;: [&#34;&lt;/strong&gt;&#34;], &#34;fields&#34;: { &#34;content&#34;: { &#34;fragment_size&#34;: 150, &#34;number_of_fragments&#34;: 3, &#34;no_match_size&#34;: 100 }, &#34;title&#34;: { &#34;number_of_fragments&#34;: 0 } } } } 参数速查 #     参数 类型 默认值 说明     pre_tags String[] [&quot;&lt;em&gt;&quot;] 高亮前标签   post_tags String[] [&quot;&lt;/em&gt;&quot;] 高亮后标签   fragment_size Integer 100 每个片段的字符数   number_of_fragments Integer 5 每字段返回的最大片段数。设为 0 返回整个字段   no_match_size Integer 0 无匹配时从字段开头返回的字符数   type String &quot;unified&quot; 高亮器类型：unified、plain、fvh   order String - 设为 &quot;score&quot; 按相关性排序片段   require_field_match Boolean true 只高亮查询匹配到的字段   encoder String &quot;default&quot; &quot;default&quot; 或 &quot;html&quot;（对 HTML 特殊字符编码）   highlight_query Object - 使用不同于搜索查询的查询来做高亮   boundary_scanner String &quot;sentence&quot;（unified）/ &quot;chars&quot;（plain/fvh） 边界扫描器：chars、word、sentence   boundary_chars String &quot;."
---


# 高亮

高亮用于在返回结果中标记命中的文本片段，提升可读性。Easysearch 在搜索阶段记录哪些文本片段匹配了查询，在返回阶段根据这些信息对原文做截取与包裹（例如 `<em>...</em>` 标签）。

## 基本用法

```json
GET /articles/_search
{
  "query": {
    "match": { "content": "搜索 引擎" }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}
```

返回结果中每条命中文档会包含 `highlight` 字段：

```json
{
  "hits": {
    "hits": [
      {
        "_source": { "content": "Easysearch 是一款高性能的搜索引擎..." },
        "highlight": {
          "content": [
            "Easysearch 是一款高性能的<em>搜索</em><em>引擎</em>..."
          ]
        }
      }
    ]
  }
}
```

## 三种高亮器类型

Easysearch 支持三种高亮器，通过 `type` 参数指定：

| 类型 | 说明 | 优缺点 |
|------|------|--------|
| `unified`（默认） | 基于 Lucene Unified Highlighter，使用 BM25 对片段评分 | 推荐首选。支持所有字段类型，性能与准确性平衡最佳 |
| `plain` | 基于标准 Lucene Highlighter，逐词高亮 | 小文本场景尚可，大文本或复杂查询时性能较差 |
| `fvh` | Fast Vector Highlighter，需要 `term_vector` 设置为 `with_positions_offsets` | 大文本高亮最快，但需要额外索引存储 |

### 使用 FVH 高亮器

```json
PUT /articles
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "term_vector": "with_positions_offsets"
      }
    }
  }
}
```

```json
GET /articles/_search
{
  "query": {
    "match": { "content": "搜索 引擎" }
  },
  "highlight": {
    "type": "fvh",
    "fields": {
      "content": {}
    }
  }
}
```

## 常用参数

```json
GET /articles/_search
{
  "query": {
    "match": { "content": "搜索 引擎" }
  },
  "highlight": {
    "pre_tags": ["<strong>"],
    "post_tags": ["</strong>"],
    "fields": {
      "content": {
        "fragment_size": 150,
        "number_of_fragments": 3,
        "no_match_size": 100
      },
      "title": {
        "number_of_fragments": 0
      }
    }
  }
}
```

### 参数速查

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `pre_tags` | String[] | `["<em>"]` | 高亮前标签 |
| `post_tags` | String[] | `["</em>"]` | 高亮后标签 |
| `fragment_size` | Integer | `100` | 每个片段的字符数 |
| `number_of_fragments` | Integer | `5` | 每字段返回的最大片段数。设为 `0` 返回整个字段 |
| `no_match_size` | Integer | `0` | 无匹配时从字段开头返回的字符数 |
| `type` | String | `"unified"` | 高亮器类型：`unified`、`plain`、`fvh` |
| `order` | String | - | 设为 `"score"` 按相关性排序片段 |
| `require_field_match` | Boolean | `true` | 只高亮查询匹配到的字段 |
| `encoder` | String | `"default"` | `"default"` 或 `"html"`（对 HTML 特殊字符编码） |
| `highlight_query` | Object | - | 使用不同于搜索查询的查询来做高亮 |
| `boundary_scanner` | String | `"sentence"`（unified）/ `"chars"`（plain/fvh） | 边界扫描器：`chars`、`word`、`sentence` |
| `boundary_chars` | String | `".,!? \t\n"` | 边界字符（与 `chars` 扫描器配合） |
| `boundary_max_scan` | Integer | `20` | 向前扫描边界字符的最大距离 |
| `boundary_scanner_locale` | String | `Locale.ROOT` | 与 `word`/`sentence` 扫描器配合使用的语言区域设置 |
| `fragmenter` | String | `"span"` | 指定片段分割方式：`span`（按查询匹配位置分割）或 `simple`（按固定大小分割）。仅适用于 `plain` 高亮器 |
| `tags_schema` | String | - | 设为 `"styled"` 使用内置的 `<em class="hltN">` 标签方案（N=1..10），替代自定义 `pre_tags`/`post_tags` |
| `max_analyzed_offset` | Integer | `1000000` | 限制分析的最大字符偏移量。超过此值的文本不会被高亮。可在索引级通过 `index.highlight.max_analyzed_offset` 配置 |
| `phrase_limit` | Integer | `256` | FVH 考虑的最大短语数 |
| `force_source` | Boolean | `false` | 强制从 `_source` 读取高亮文本 |
| `matched_fields` | String[] | - | FVH 专用：合并多个字段的匹配做高亮 |

## 多字段高亮

不同字段可以使用不同的高亮配置：

```json
GET /articles/_search
{
  "query": {
    "multi_match": {
      "query": "分布式 搜索",
      "fields": ["title^3", "content"]
    }
  },
  "highlight": {
    "pre_tags": ["<mark>"],
    "post_tags": ["</mark>"],
    "fields": {
      "title": {
        "number_of_fragments": 0
      },
      "content": {
        "fragment_size": 200,
        "number_of_fragments": 3,
        "order": "score"
      }
    }
  }
}
```

> `title` 设置 `number_of_fragments: 0` 表示返回整个字段内容并高亮，适合短字段。

## 使用自定义查询做高亮

有时你希望搜索和高亮使用不同的查询条件：

```json
GET /articles/_search
{
  "query": {
    "bool": {
      "must": { "match": { "content": "搜索" } },
      "filter": { "term": { "status": "published" } }
    }
  },
  "highlight": {
    "fields": {
      "content": {
        "highlight_query": {
          "match": { "content": "搜索 引擎 分布式" }
        }
      }
    }
  }
}
```

## 性能考量

高亮不是免费的：

- `unified`（默认）性能最好，适合大多数场景
- `fvh` 对大文本最快，但需要 `term_vector: with_positions_offsets`，会增加索引大小约 30–50%
- `plain` 对大文本性能最差，应避免在大文本字段上使用

建议：

- 只对需要展示的字段做高亮，避免全字段高亮
- 控制 `fragment_size` 和 `number_of_fragments`，避免返回过大的高亮结果
- 对超长文本，考虑在建模时预先准备"摘要字段"，在摘要上做高亮
- 高并发场景优先使用 `unified`

## 常见问题

- **匹配到结果但没有高亮**：检查查询字段与高亮字段是否一致（如 `content` vs `content.keyword`），检查字段 analyzer 是否匹配
- **高亮片段截断不理想**：调整 `fragment_size`，或使用 `boundary_scanner: "sentence"` 按句子切分
- **中文高亮分词不对**：确保高亮字段和查询使用相同的分词器（如 IK 分词器）
- **HTML 标签被转义**：使用 `"encoder": "html"` 对结果中的 HTML 特殊字符编码

## 小结

- 高亮用于在返回结果中标记命中的文本片段，提升可读性
- 三种高亮器：`unified`（推荐）、`plain`、`fvh`（大文本最快）
- 可以为不同字段配置不同的高亮选项
- 控制高亮字段数量和片段大小，关注性能影响

下一步可以继续阅读：

- [建议与纠错](./suggestions.md)
- [分页与排序](./pagination-and-sorting.md)
- [相关性](./relevance/_index.md)

## 参考手册（API 与参数）

- [高亮参数详解（功能手册）]({{< relref "/docs/features/query-dsl/highlight.md" >}})



