---
title: "高亮"
date: 0001-01-01
description: "高亮命中词的 API 参数、高亮器类型选择、自定义标签、片段控制。"
summary: "高亮 #  高亮用于在返回结果中突出显示命中的查询词，方便用户快速定位关键信息。
相关指南 #    高亮   基础用法 #  在查询体中添加 highlight 段即可启用：
GET shakespeare/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;text_entry&#34;: &#34;life&#34; } }, &#34;highlight&#34;: { &#34;fields&#34;: { &#34;text_entry&#34;: {} } } } 响应中每条命中文档会带一个 highlight 对象，默认使用 &lt;em&gt; 标签包裹：
&#34;highlight&#34;: { &#34;text_entry&#34;: [ &#34;my &lt;em&gt;life&lt;/em&gt;, except my &lt;em&gt;life&lt;/em&gt;.&#34; ] }  自定义标签 #  通过 pre_tags / post_tags 自定义包裹标签：
GET shakespeare/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;play_name&#34;: &#34;Henry IV&#34; } }, &#34;highlight&#34;: { &#34;pre_tags&#34;: [&#34;&lt;strong&gt;&#34;], &#34;post_tags&#34;: [&#34;&lt;/strong&gt;&#34;], &#34;fields&#34;: { &#34;play_name&#34;: {} } } } 也可以使用 &quot;tags_schema&quot;: &quot;styled&quot; 启用内置的多级标签样式（&lt;em class=&quot;hlt1&quot;&gt;、&lt;em class=&quot;hlt2&quot;&gt; 等）。"
---


# 高亮

高亮用于在返回结果中突出显示命中的查询词，方便用户快速定位关键信息。

## 相关指南

- [高亮]({{< relref "/docs/features/fulltext-search/highlighting.md" >}})

---

## 基础用法

在查询体中添加 `highlight` 段即可启用：

```json
GET shakespeare/_search
{
  "query": {
    "match": { "text_entry": "life" }
  },
  "highlight": {
    "fields": {
      "text_entry": {}
    }
  }
}
```

响应中每条命中文档会带一个 `highlight` 对象，默认使用 `<em>` 标签包裹：

```json
"highlight": {
  "text_entry": [
    "my <em>life</em>, except my <em>life</em>."
  ]
}
```

---

## 自定义标签

通过 `pre_tags` / `post_tags` 自定义包裹标签：

```json
GET shakespeare/_search
{
  "query": {
    "match": { "play_name": "Henry IV" }
  },
  "highlight": {
    "pre_tags": ["<strong>"],
    "post_tags": ["</strong>"],
    "fields": {
      "play_name": {}
    }
  }
}
```

也可以使用 `"tags_schema": "styled"` 启用内置的多级标签样式（`<em class="hlt1">`、`<em class="hlt2">` 等）。

---

## 高亮器类型（type）

Easysearch 支持三种高亮器，通过 `type` 参数指定：

| 类型 | 说明 | 特点 |
|------|------|------|
| `unified` | **默认**，基于 Lucene Unified Highlighter | 性能好，支持所有字段类型 |
| `plain` | 标准高亮器 | 精确度高，适合小字段 |
| `fvh` | Fast Vector Highlighter | 需要 `term_vector` 设置，适合大文本字段 |

```json
GET shakespeare/_search
{
  "query": {
    "match": { "text_entry": "life" }
  },
  "highlight": {
    "fields": {
      "text_entry": {
        "type": "unified"
      }
    }
  }
}
```

### 选择建议

| 场景 | 推荐类型 |
|------|---------|
| 通用场景 | `unified`（默认即可） |
| 需要精确片段控制 | `plain` |
| 大文本字段（如文章正文） | `fvh`（需在 mapping 中设置 `"term_vector": "with_positions_offsets"`） |

---

## 片段控制参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `fragment_size` | 每个高亮片段的最大字符数 | 100 |
| `number_of_fragments` | 返回的最大片段数量 | 5 |
| `no_match_size` | 无匹配时返回的字段内容长度（0 表示不返回） | 0 |
| `fragment_offset` | 片段的起始偏移量（仅 `fvh`） | - |
| `order` | 片段排序方式，`score` 表示按相关性排序 | 无特定排序 |

```json
GET shakespeare/_search
{
  "query": {
    "match": { "text_entry": "life" }
  },
  "highlight": {
    "fields": {
      "text_entry": {
        "fragment_size": 150,
        "number_of_fragments": 3,
        "no_match_size": 50
      }
    }
  }
}
```

---

## 高级参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `require_field_match` | 是否只高亮查询中指定的字段 | true |
| `highlight_query` | 使用不同于主查询的查询来生成高亮 | 主查询 |
| `matched_fields` | 组合多个字段的匹配来生成高亮（仅 `fvh`） | - |
| `phrase_limit` | 短语匹配的最大数量（仅 `fvh`） | 256 |
| `encoder` | 输出编码方式 | `default`（原文）或 `html`（HTML 转义） |

### 边界扫描参数（boundary_scanner）

控制高亮片段如何在句子/段落边界处截断：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `boundary_scanner` | 边界扫描模式 | `chars`（字符级）、`sentence`（句子级）、`word`（词级） |
| `boundary_chars` | 作为边界的字符集 | `.,!? \t\n` |
| `boundary_max_scan` | 扫描边界字符的最大范围 | 20 |
| `boundary_scanner_locale` | 句子/词级分割的语言区域 | 系统默认 |

```json
GET shakespeare/_search
{
  "query": {
    "match": { "text_entry": "question" }
  },
  "highlight": {
    "fields": {
      "text_entry": {
        "boundary_scanner": "sentence",
        "fragment_size": 200
      }
    }
  }
}
```

---

## 多字段高亮

可以同时高亮多个字段，每个字段可以有不同的配置：

```json
GET shakespeare/_search
{
  "query": {
    "multi_match": {
      "query": "king",
      "fields": ["play_name", "text_entry"]
    }
  },
  "highlight": {
    "fields": {
      "play_name": {
        "number_of_fragments": 0
      },
      "text_entry": {
        "fragment_size": 150,
        "number_of_fragments": 3
      }
    }
  }
}
```

> **提示**：`number_of_fragments` 设为 `0` 时返回整个字段内容而非片段，适合短字段（如标题）。

> **同义词/词干**：即便查询时启用了同义词或词干还原，高亮依然会基于原始文本中的命中词来生成片段，保证展示对用户友好。

