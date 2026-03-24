---
title: "邻近匹配"
date: 0001-01-01
description: "短语匹配、slop、多值字段处理与位置感知查询。"
summary: "邻近匹配 #  标准全文检索可以把字段视为“一袋词”（bag of words）：match 能告诉你这些词是否存在，但无法表达词与词之间的顺序与距离。这会导致一些明显不合理的匹配：
 Sue ate the alligator. The alligator ate Sue. Sue never goes anywhere without her alligator-skin purse.  搜索 sue alligator 时，上面三句都会被 match 命中，但它们的语义完全不同。邻近匹配（proximity matching）并不能“理解语义”，但它能利用位置信息来判断词项是否相邻、是否接近，从而让结果更符合直觉。
match_phrase：短语匹配 #  match_phrase 是最常用的位置敏感查询：要求词项按顺序出现，并且位置相邻。
GET /my_index/_search { &#34;query&#34;: { &#34;match_phrase&#34;: { &#34;title&#34;: &#34;quick brown fox&#34; } } } 它的核心逻辑是：查询字符串先被分析为词项列表，然后只保留那些同时包含全部词项，且词项位置关系一致的文档。
词项位置（position）从哪来？ #  分词不仅会产出 tokens，还会产出 position。你可以用 _analyze 观察：
GET /_analyze { &#34;analyzer&#34;: &#34;standard&#34;, &#34;text&#34;: &#34;Quick brown fox&#34; } 典型输出会包含 position（示意）："
---


# 邻近匹配

标准全文检索可以把字段视为“一袋词”（bag of words）：`match` 能告诉你这些词是否存在，但无法表达词与词之间的**顺序与距离**。这会导致一些明显不合理的匹配：

- `Sue ate the alligator.`
- `The alligator ate Sue.`
- `Sue never goes anywhere without her alligator-skin purse.`

搜索 `sue alligator` 时，上面三句都会被 `match` 命中，但它们的语义完全不同。邻近匹配（proximity matching）并不能“理解语义”，但它能利用**位置信息**来判断词项是否相邻、是否接近，从而让结果更符合直觉。

## match_phrase：短语匹配

`match_phrase` 是最常用的位置敏感查询：要求词项按顺序出现，并且位置相邻。

```json
GET /my_index/_search
{
  "query": {
    "match_phrase": {
      "title": "quick brown fox"
    }
  }
}
```

它的核心逻辑是：查询字符串先被分析为词项列表，然后只保留那些同时包含全部词项，且词项**位置关系一致**的文档。

### 词项位置（position）从哪来？

分词不仅会产出 tokens，还会产出 position。你可以用 `_analyze` 观察：

```json
GET /_analyze
{
  "analyzer": "standard",
  "text": "Quick brown fox"
}
```

典型输出会包含 `position`（示意）：

- `quick`：position 1
- `brown`：position 2
- `fox`：position 3

短语 `quick brown fox` 能匹配的条件就是：

- 三个词都出现
- `brown.position = quick.position + 1`
- `fox.position = quick.position + 2`

> 说明：从底层看，短语匹配本质上会落到更底层的 span 家族能力上，但绝大多数场景使用 `match_phrase` 即可。

## slop：让短语匹配“没那么严格”

精确短语过于严格时，可以用 `slop` 允许“插词/换位”等轻微偏离：

```json
GET /my_index/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "quick fox",
        "slop": 1
      }
    }
  }
}
```

`slop` 可以理解为：为了让查询词序与文档对齐，允许“移动词项”多少步。`slop` 越大，越宽松，也越容易引入噪声与性能开销。

## 多值字段与 position_increment_gap

权威指南里有一个非常经典的坑：数组字段的每个元素在分析后会被“连续拼接”，导致跨元素短语被意外命中。

示例文档：

```json
PUT /my_index/_doc/1
{
  "names": [ "John Abraham", "Lincoln Smith" ]
}
```

短语查询：

```json
GET /my_index/_search
{
  "query": {
    "match_phrase": {
      "names": "Abraham Lincoln"
    }
  }
}
```

这可能会命中，因为 `abraham` 与 `lincoln` 在 position 上是相邻的（一个来自第一个数组元素，一个来自第二个）。

解决方案是给数组元素之间留“足够大的空隙”：`position_increment_gap`：

```json
PUT /my_index/_mapping
{
  "properties": {
    "names": {
      "type": "text",
      "position_increment_gap": 100
    }
  }
}
```

这样第二个元素的词项会被整体平移 100 个 position；要跨元素做短语匹配就必须设置一个极大的 `slop`，从而默认情况下避免误命中。

## 性能：短语/邻近查询为什么更贵？

`match` 只需要判断词项是否存在于倒排索引；`match_phrase` 还需要读取 positions 并做位置组合计算。权威指南给出的直觉是：

- term 查询远快于短语查询
- 有 `slop` 的邻近查询更贵

不过在常见文本场景里，短语查询通常仍然是可用的（毫秒级别很常见）。真正麻烦的是某些“病理数据”（例如大量重复词项的序列）叠加很大的 `slop` 值，会导致组合爆炸。

### 只对 top-N 结果做邻近增强：rescore

一个很实用的性能策略是：用便宜的 `match` 先找候选并排序，再用更贵的短语/邻近查询只对每分片 top-K 做重新评分（rescore）。

```json
GET /my_index/_search
{
  "query": {
    "match": {
      "title": {
        "query": "quick brown fox",
        "minimum_should_match": "30%"
      }
    }
  },
  "rescore": {
    "window_size": 50,
    "query": {
      "rescore_query": {
        "match_phrase": {
          "title": {
            "query": "quick brown fox",
            "slop": 50
          }
        }
      }
    }
  }
}
```

直觉上：`match` 决定“进候选集的人”，rescore 决定“候选集里谁排得更前”。

## shingles：索引时的“词对信号”，更灵活也更快

短语/邻近查询严格要求词项存在并且靠近。若你希望捕捉“关联词对”并提升排序，而不是严格过滤，可以在索引时生成 shingles（常见是 bigram）。

对句子 `Sue ate the alligator`：

- unigrams：`sue`、`ate`、`the`、`alligator`
- bigrams：`sue ate`、`ate the`、`the alligator`

这样你可以：

- 用 unigram 字段作为基础召回（必须）
- 用 bigram 字段作为加分信号（should）

### 生成 shingles（示意）

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_shingle_filter": {
          "type": "shingle",
          "min_shingle_size": 2,
          "max_shingle_size": 2,
          "output_unigrams": false
        }
      },
      "analyzer": {
        "my_shingle_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [ "lowercase", "my_shingle_filter" ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "shingles": {
            "type": "text",
            "analyzer": "my_shingle_analyzer"
          }
        }
      }
    }
  }
}
```

### 查询：unigram 负责召回，shingles 负责加分

```json
GET /my_index/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "title": "the hungry alligator ate sue"
        }
      },
      "should": {
        "match": {
          "title.shingles": "the hungry alligator ate sue"
        }
      }
    }
  }
}
```

权威指南的经验是：shingles 查询像普通 `match` 一样高效（查询时），而短语/邻近查询需要 positions 计算。代价转移到了索引端（更多词项、更多磁盘）。

## 小结

- `match_phrase` 使用 positions 做严格短语匹配；`slop` 提供可控的宽松度
- 数组字段短语匹配要注意跨元素误命中，用 `position_increment_gap` 做隔离
- 性能上可采用“先 match、后 rescore”的两阶段思路，把昂贵计算只用于 top-N
- shingles 是索引时策略：用 bigram 作为相关性信号，通常更灵活也更快

下一步可以继续阅读：

- [部分匹配](./partial-matching.md)
- [多字段搜索](./multi-field-search.md)
- [文本分析：同义词](../mapping-and-analysis/text-analysis-synonyms.md)

## 参考手册（API 与参数）

- [短语匹配（match_phrase）（功能手册）]({{< relref "/docs/features/fulltext-search/full-text/match-phrase.md" >}})
- [短语前缀匹配（match_phrase_prefix）（功能手册）]({{< relref "/docs/features/fulltext-search/full-text/match-phrase-prefix.md" >}})
- [Intervals 查询（功能手册）]({{< relref "/docs/features/fulltext-search/full-text/intervals.md" >}})

