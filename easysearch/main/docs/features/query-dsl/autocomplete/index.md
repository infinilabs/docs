---
title: "自动补全"
date: 0001-01-01
description: "前缀匹配、Edge N-gram、Completion Suggester 等自动补全实现方式的 API 与参数说明。"
summary: "自动补全 #  自动补全是在用户输入过程中，实时给出可能的搜索词建议。Easysearch 支持三种实现方式，各有适用场景：
   方式 时机 性能 适用场景     前缀匹配（match_phrase_prefix） 查询时 一般 快速原型，无需特殊 mapping   Edge N-gram 索引时 好 大规模数据的前缀补全   Completion Suggester 索引时 最优 高并发自动补全，支持权重控制    相关指南 #    建议与纠错  部分匹配   前缀匹配（match_phrase_prefix） #  前缀匹配会在查询时，对最后一个 term 做前缀展开。典型做法是使用 match_phrase_prefix 在 text 字段上做查询。
不需要特殊 mapping，可以直接在现有 text 字段上使用。
GET shakespeare/_search { &#34;query&#34;: { &#34;match_phrase_prefix&#34;: { &#34;text_entry&#34;: { &#34;query&#34;: &#34;qui&#34;, &#34;slop&#34;: 3 } } } } 参数说明 #     参数 说明 默认值     query 查询文本 必填   slop 允许的词项位置偏移量 0   max_expansions 最后一个词项的最大前缀展开数量 50   analyzer 覆盖默认分析器 字段默认分析器   zero_terms_query 当分析器移除所有词项时的行为（none 或 all） none     性能注意：前缀匹配属于相对昂贵的查询。例如前缀为 a 时，可能会匹配到几十万 terms。建议通过 max_expansions 限制展开规模："
---


# 自动补全

自动补全是在用户输入过程中，实时给出可能的搜索词建议。Easysearch 支持三种实现方式，各有适用场景：

| 方式 | 时机 | 性能 | 适用场景 |
|------|------|------|---------|
| 前缀匹配（match_phrase_prefix） | 查询时 | 一般 | 快速原型，无需特殊 mapping |
| Edge N-gram | 索引时 | 好 | 大规模数据的前缀补全 |
| Completion Suggester | 索引时 | 最优 | 高并发自动补全，支持权重控制 |

## 相关指南

- [建议与纠错]({{< relref "/docs/features/fulltext-search/suggestions.md" >}})
- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})

---

## 前缀匹配（match_phrase_prefix）

前缀匹配会在查询时，对最后一个 term 做前缀展开。典型做法是使用 `match_phrase_prefix` 在 `text` 字段上做查询。

**不需要特殊 mapping**，可以直接在现有 `text` 字段上使用。

```json
GET shakespeare/_search
{
  "query": {
    "match_phrase_prefix": {
      "text_entry": {
        "query": "qui",
        "slop": 3
      }
    }
  }
}
```

### 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `query` | 查询文本 | 必填 |
| `slop` | 允许的词项位置偏移量 | 0 |
| `max_expansions` | 最后一个词项的最大前缀展开数量 | 50 |
| `analyzer` | 覆盖默认分析器 | 字段默认分析器 |
| `zero_terms_query` | 当分析器移除所有词项时的行为（`none` 或 `all`） | `none` |

> **性能注意**：前缀匹配属于相对昂贵的查询。例如前缀为 `a` 时，可能会匹配到几十万 terms。建议通过 `max_expansions` 限制展开规模：

```json
GET shakespeare/_search
{
  "query": {
    "match_phrase_prefix": {
      "text_entry": {
        "query": "qui",
        "slop": 3,
        "max_expansions": 10
      }
    }
  }
}
```

---

## Edge N-gram 匹配

Edge N-gram 在**索引时**把单词切成一系列前缀片段，加速前缀查询。例如对 `"quick"` 做 edge N-gram：

| 前缀 | Token |
|------|-------|
| 1 字符 | `q` |
| 2 字符 | `qu` |
| 3 字符 | `qui` |
| 4 字符 | `quic` |
| 5 字符 | `quick` |

### 配置分析器

定义带 `edge_ngram` 过滤器的自定义分析器：

```json
PUT shakespeare
{
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "text",
        "analyzer": "autocomplete",
        "search_analyzer": "standard"
      }
    }
  },
  "settings": {
    "analysis": {
      "filter": {
        "edge_ngram_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "edge_ngram_filter"]
        }
      }
    }
  }
}
```

> **重要**：建议在 mapping 中指定 `search_analyzer` 为 `standard`，避免查询词本身也被切成前缀导致过多无关匹配。这是少数需要"索引时分析器 ≠ 查询时分析器"的场景之一。

### Edge N-gram 过滤器参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `min_gram` | 最小前缀长度 | 1 |
| `max_gram` | 最大前缀长度 | 2 |
| `preserve_original` | 是否保留原始 token | false |

> **提示**：除了 `edge_ngram` **过滤器**，Easysearch 也支持 `edge_ngram` **分词器**（tokenizer），它按字符级别切分而非基于词级别。过滤器在大多数自动补全场景中更实用。

### 测试分析器

```json
POST shakespeare/_analyze
{
  "analyzer": "autocomplete",
  "text": "quick"
}
```

### 查询示例

```json
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": {
        "query": "qui",
        "analyzer": "standard"
      }
    }
  }
}
```

---

## Completion Suggester

Completion Suggester 使用基于 **FST（有限状态转换器）** 的数据结构，常驻内存、专门优化前缀匹配性能，适合高并发自动补全场景。

### 创建 Mapping

使用专门的 `completion` 字段类型：

```json
PUT shakespeare
{
  "mappings": {
    "properties": {
      "text_entry": {
        "type": "completion"
      }
    }
  }
}
```

### 基础查询

```json
GET shakespeare/_search
{
  "suggest": {
    "autocomplete": {
      "prefix": "To be",
      "completion": {
        "field": "text_entry",
        "size": 5
      }
    }
  }
}
```

### Completion 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `prefix` | 前缀文本 | - |
| `field` | completion 字段名 | 必填 |
| `size` | 返回建议数量 | 5 |
| `skip_duplicates` | 过滤重复建议 | false |
| `fuzzy` | 启用模糊补全（容错输入） | 不启用 |

### 模糊补全

支持容错输入的自动补全，用户打错字也能返回正确建议：

```json
GET shakespeare/_search
{
  "suggest": {
    "autocomplete": {
      "prefix": "To bo",
      "completion": {
        "field": "text_entry",
        "fuzzy": {
          "fuzziness": "AUTO"
        }
      }
    }
  }
}
```

#### fuzzy 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `fuzziness` | 编辑距离（`AUTO`、`0`、`1`、`2`） | `AUTO` |
| `transpositions` | 是否允许字符交换 | true |
| `min_length` | 开始模糊匹配的最小输入长度 | 3 |
| `prefix_length` | 不参与模糊的前缀字符数 | 1 |
| `unicode_aware` | 是否按 Unicode 字符计算编辑距离 | false |

### 正则补全

支持基于正则表达式的补全匹配：

```json
GET shakespeare/_search
{
  "suggest": {
    "autocomplete": {
      "regex": "To b[aeiou].*",
      "completion": {
        "field": "text_entry"
      }
    }
  }
}
```

### 权重控制

通过 `input` 和 `weight` "运营"特定建议词：

```json
PUT shakespeare/_doc/1
{
  "text_entry": {
    "input": ["To be, or not to be: that is the question:"],
    "weight": 10
  }
}
```

权重更高的候选会优先排在补全结果前面。

---

## Term Suggester（拼写纠正）

Term Suggester 基于编辑距离对单个词项提供拼写纠正建议：

```json
GET shakespeare/_search
{
  "suggest": {
    "spell-check": {
      "text": "lief",
      "term": {
        "field": "text_entry"
      }
    }
  }
}
```

返回按 `score` 排序的纠正候选，`freq` 表示该词在索引中出现的次数。分数越高建议越好。

---

## Phrase Suggester（短语纠正）

Phrase Suggester 使用 N-gram 语言模型对整个短语进行纠正，实现"Did you mean ...?"功能。

需要先配置 `shingle` 过滤器来构建词级 N-gram：

```json
PUT shakespeare
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
      "text_entry": {
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

使用 phrase 建议：

```json
POST shakespeare/_search
{
  "suggest": {
    "text": "That the qution",
    "simple_phrase": {
      "phrase": {
        "field": "text_entry.trigram"
      }
    }
  }
}
```

返回纠正后的短语：`"that is the question"`。

