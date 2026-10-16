---
title: "建议与纠错"
date: 0001-01-01
description: "Term Suggester、Phrase Suggester、Completion Suggester 的用法、参数与常见场景。"
summary: "建议与纠错 #  Easysearch 提供三种 Suggester（建议器），覆盖&quot;拼写纠正→短语纠正→前缀补全&quot;三个典型场景：
   Suggester 用途 工作方式     Term Suggester 单词拼写纠正 基于编辑距离，对每个词项找索引中相似的词   Phrase Suggester 短语级纠正（&ldquo;Did you mean …?&quot;） 使用 N-gram 语言模型对整个短语做纠正   Completion Suggester 前缀自动补全 基于 FST 数据结构常驻内存，毫秒级响应    所有 Suggester 通过 _search 请求的 suggest 参数调用，可以与 query 同时使用。
Term Suggester（单词纠正） #  对单个词项，基于编辑距离在索引中查找相似候选词。
GET /articles/_search { &#34;suggest&#34;: { &#34;spell-check&#34;: { &#34;text&#34;: &#34;quer&#34;, &#34;term&#34;: { &#34;field&#34;: &#34;content&#34;, &#34;suggest_mode&#34;: &#34;popular&#34; } } } } 返回示例："
---


# 建议与纠错

Easysearch 提供三种 Suggester（建议器），覆盖"拼写纠正→短语纠正→前缀补全"三个典型场景：

| Suggester | 用途 | 工作方式 |
|-----------|------|----------|
| Term Suggester | 单词拼写纠正 | 基于编辑距离，对每个词项找索引中相似的词 |
| Phrase Suggester | 短语级纠正（"Did you mean …?"） | 使用 N-gram 语言模型对整个短语做纠正 |
| Completion Suggester | 前缀自动补全 | 基于 FST 数据结构常驻内存，毫秒级响应 |

所有 Suggester 通过 `_search` 请求的 `suggest` 参数调用，可以与 `query` 同时使用。

## Term Suggester（单词纠正）

对单个词项，基于编辑距离在索引中查找相似候选词。

```json
GET /articles/_search
{
  "suggest": {
    "spell-check": {
      "text": "quer",
      "term": {
        "field": "content",
        "suggest_mode": "popular"
      }
    }
  }
}
```

返回示例：

```json
{
  "suggest": {
    "spell-check": [
      {
        "text": "quer",
        "options": [
          { "text": "query", "score": 0.75, "freq": 128 },
          { "text": "quer", "score": 0.5, "freq": 2 }
        ]
      }
    ]
  }
}
```

### Term Suggester 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `field` | String | 必填 | 从哪个字段获取候选词 |
| `suggest_mode` | String | `missing` | `missing`：仅在原词不存在时建议；`popular`：建议频率更高的词；`always`：总是返回建议 |
| `accuracy` | Float | `0.5` | 候选词的最低相似度阈值（0.0~1.0） |
| `sort` | String | `score` | `score`：先按相似度排序；`frequency`：先按词频排序 |
| `max_edits` | Integer | `2` | 最大编辑距离（只能是 1 或 2） |
| `prefix_length` | Integer | `1` | 前缀必须匹配的字符数，值越大候选越少但速度越快 |
| `min_word_length` | Integer | `4` | 原词长度低于此值时不返回建议 |
| `min_doc_freq` | Float/Integer | `0` | 候选词的最低文档频率（绝对值或比例），过滤罕见词 |
| `max_term_freq` | Float | `0.01` | 原词频率高于此比例时不返回建议（高频词一般无需纠正） |
| `string_distance` | String | `internal` | 相似度算法：`internal`、`damerau_levenshtein`、`levenshtein`、`jaro_winkler`、`ngram` |
| `max_inspections` | Integer | `5` | 检查候选词的倍数因子（越大结果越好但越慢） |

> **建议**：在搜索结果为空或很少时触发 Term Suggester，搭配 `suggest_mode: "popular"` 能给出更有价值的建议。

## Phrase Suggester（短语纠正）

对整个短语做语言模型级纠正，实现"您是不是要找：xxx？"功能。相比 Term Suggester，Phrase Suggester 考虑词与词之间的关系。

**前提**：需要用 `shingle`（词级 N-gram）过滤器建立索引：

```json
PUT /articles
{
  "settings": {
    "analysis": {
      "analyzer": {
        "trigram": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "shingle"]
        }
      },
      "filter": {
        "shingle": {
          "type": "shingle",
          "min_shingle_size": 2,
          "max_shingle_size": 3
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "fields": {
          "trigram": {
            "type": "text",
            "analyzer": "trigram"
          }
        }
      }
    }
  }
}
```

查询：

```json
GET /articles/_search
{
  "suggest": {
    "phrase-correction": {
      "text": "full texr serch",
      "phrase": {
        "field": "content.trigram",
        "confidence": 1.0,
        "max_errors": 2,
        "direct_generator": [
          {
            "field": "content.trigram",
            "suggest_mode": "popular"
          }
        ]
      }
    }
  }
}
```

### Phrase Suggester 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `field` | String | 必填 | shingle 字段名 |
| `max_errors` | Float | `1.0` | 允许纠正的最大词项数（可以是比例，如 `0.5` 表示一半的词可以被纠正） |
| `confidence` | Float | `1.0` | 置信度阈值，只返回得分高于原输入 × confidence 的建议。设为 `0` 返回所有候选 |
| `separator` | String | `" "` | 词项之间的分隔符 |
| `gram_size` | Integer | - | N-gram 大小，默认由 `shingle_size` 推断 |
| `real_word_error_likelihood` | Float | `0.95` | 真实词被拼错的概率，降低此值会让纠正倾向于不修改真实存在的词 |
| `force_unigrams` | Boolean | `true` | 是否强制使用 unigram 作为候选 |
| `token_limit` | Integer | `10` | 最多分析的 token 数量 |
| `smoothing` | Object | - | 平滑模型：`stupid_backoff`（默认）、`laplace`、`linear_interpolation` |
| `highlight` | Object | - | 在建议结果中高亮被修改的词（设置 `pre_tag` 和 `post_tag`） |
| `collate` | Object | - | 用模板查询校验建议是否确实能命中文档，可设置 `prune: true` 保留未命中的建议 |

### direct_generator 参数

`direct_generator` 是 Phrase Suggester 生成候选纠正词的核心组件，支持以下参数：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `field` | String | 必填 | 从哪个字段获取候选词 |
| `suggest_mode` | String | `"missing"` | 建议模式：`missing`（仅当原词不在索引中时建议）、`popular`（建议更常见的替代词）、`always`（总是建议） |
| `size` | Integer | `5` | 每个 token 返回的最大候选数 |
| `max_edits` | Integer | `2` | 允许的最大编辑距离（1 或 2） |
| `prefix_length` | Integer | `1` | 前缀不参与模糊匹配的字符数 |
| `min_word_length` | Integer | `4` | 候选词的最小长度 |
| `max_inspections` | Integer | `5` | 每个候选词的最大检查次数，用于控制性能 |
| `min_doc_freq` | Float | `0` | 候选词在索引中的最低文档频率（绝对值或比例） |
| `max_term_freq` | Float | `0.01` | 原始词的最大词频阈值，超过此值不做纠正 |
| `pre_filter` | String | - | 应用于候选词的分析过滤器 |
| `post_filter` | String | - | 应用于候选词的后置过滤器 |

### Phrase Suggester 高亮

可以在纠正结果中高亮被修改的词：

```json
GET /articles/_search
{
  "suggest": {
    "phrase-correction": {
      "text": "full texr serch",
      "phrase": {
        "field": "content.trigram",
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        }
      }
    }
  }
}
```

返回：`"full <em>text</em> <em>search</em>"`

## Completion Suggester（前缀补全）

基于 FST 数据结构常驻内存，为高并发前缀补全场景做了专门优化。

**前提**：使用 `completion` 字段类型：

```json
PUT /search-terms
{
  "mappings": {
    "properties": {
      "suggest": {
        "type": "completion"
      }
    }
  }
}
```

写入带权重的数据：

```json
PUT /search-terms/_doc/1
{
  "suggest": {
    "input": ["全文搜索", "全文检索", "全文匹配"],
    "weight": 10
  }
}

PUT /search-terms/_doc/2
{
  "suggest": {
    "input": ["前缀补全", "前缀匹配"],
    "weight": 5
  }
}
```

查询：

```json
GET /search-terms/_search
{
  "suggest": {
    "autocomplete": {
      "prefix": "全文",
      "completion": {
        "field": "suggest",
        "size": 5,
        "skip_duplicates": true
      }
    }
  }
}
```

### 模糊补全

允许用户输入有拼写错误：

```json
GET /search-terms/_search
{
  "suggest": {
    "autocomplete": {
      "prefix": "全问",
      "completion": {
        "field": "suggest",
        "fuzzy": {
          "fuzziness": "AUTO",
          "transpositions": true,
          "min_length": 3,
          "prefix_length": 1,
          "unicode_aware": false
        }
      }
    }
  }
}
```

### 正则补全

使用正则表达式匹配：

```json
GET /search-terms/_search
{
  "suggest": {
    "autocomplete": {
      "regex": "全[文本].*",
      "completion": {
        "field": "suggest"
      }
    }
  }
}
```

### Completion Suggester 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `field` | String | 必填 | completion 字段名 |
| `size` | Integer | `5` | 返回建议的数量 |
| `skip_duplicates` | Boolean | `false` | 过滤重复的建议文本 |
| `fuzzy` | Object | - | 启用模糊补全，容错输入。见下方 fuzzy 参数 |
| `regex` | String | - | 使用正则表达式匹配（替代 `prefix`） |
| `contexts` | Object | - | 上下文过滤（需要字段配置 `contexts`），按类目或地理位置过滤建议 |

## 常见场景与选型

| 场景 | 推荐方案 | 说明 |
|------|---------|------|
| 搜索框下拉补全 | Completion Suggester | FST 常驻内存，最快 |
| "您是不是要找" | Phrase Suggester | 整句纠正，用户体验好 |
| 搜索结果为空时纠错 | Term Suggester | 简单场景，无需额外 mapping |
| 前缀匹配快速原型 | match_phrase_prefix | 无需特殊 mapping，但性能一般 |
| 大规模前缀补全 | Edge N-gram + match | 索引时切分，查询时性能好 |

## 实践建议

- **从简单开始**：先用 Completion Suggester 实现搜索框补全，效果明显且性能最好
- **纠错按需启用**：Term / Phrase Suggester 适合在搜索结果为空时触发，避免过度纠错覆盖用户真实意图
- **权重运营**：通过 `weight` 字段控制热门搜索词的排序，结合业务数据定期更新
- **候选词清洗**：建议词来源（搜索日志、商品标题等）需要去重、过滤敏感词
- **所有建议逻辑都应遵循：宁可少给、不给，也不要给出明显错误的建议**

## 小结

- Easysearch 提供三种 Suggester：Term（单词纠正）、Phrase（短语纠正）、Completion（前缀补全）
- Completion Suggester 性能最优，适合搜索框实时补全
- Phrase Suggester 适合实现"您是不是要找"功能，需要 shingle 索引
- Term Suggester 最简单，适合基础拼写纠错

下一步可以继续阅读：

- [高亮](./highlighting.md)
- [分页与排序](./pagination-and-sorting.md)
- [相关性](./relevance/_index.md)

## 参考手册（API 与参数）

- [自动补全与建议（功能手册）]({{< relref "/docs/features/query-dsl/autocomplete.md" >}})




