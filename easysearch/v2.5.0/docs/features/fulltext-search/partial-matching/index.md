---
title: "部分匹配"
date: 0001-01-01
description: "前缀查询、通配符、正则表达式、match_phrase_prefix 等部分匹配查询。"
summary: "部分匹配 #  部分匹配允许用户输入“词的一部分”，并找出包含该片段的词项。全文检索很多时候并不需要它（分析器已经做了词干/同义词等处理），但在权威指南总结的几类场景中，部分匹配非常有价值：
 精确值字段：邮编、产品序列号等以固定前缀/模式出现的值 输入即搜索（search-as-you-type）：用户还没输完就给出候选结果 复合词语言：德语/荷兰语等将多个词组合成长词  本页按“查询时方案 → 索引时优化”的顺序展开，并重点强调性能与可控性。
查询时方案 1：prefix 前缀查询 #  查找所有以 W1 开头的邮编：
GET /my_index/_search { &#34;query&#34;: { &#34;prefix&#34;: { &#34;postcode&#34;: &#34;W1&#34; } } } prefix 是词项级查询，不会分析输入，它做的事情近似于：
 扫描倒排词典（有序词列表），找到第一个以 W1 开头的词 收集该词对应的文档 ID 向后移动，继续收集所有以 W1 开头的词，直到遇到不再匹配的词  重要的性能结论（权威指南强调）：前缀越短，扫描的词越多；当唯一词很多时，前缀查询的伸缩性并不好。尽量使用更长的前缀，或者转向索引时优化方案（见下文）。
查询时方案 2：wildcard / regexp（更灵活，也更危险） #  通配符查询（? 匹配一个字符，* 匹配 0 或多个字符）：
GET /my_index/_search { &#34;query&#34;: { &#34;wildcard&#34;: { &#34;postcode&#34;: &#34;W?F*HW&#34; } } } 正则表达式查询：
GET /my_index/_search { &#34;query&#34;: { &#34;regexp&#34;: { &#34;postcode&#34;: &#34;W[0-9]."
---


# 部分匹配

部分匹配允许用户输入“词的一部分”，并找出包含该片段的词项。全文检索很多时候并不需要它（分析器已经做了词干/同义词等处理），但在权威指南总结的几类场景中，部分匹配非常有价值：

- **精确值字段**：邮编、产品序列号等以固定前缀/模式出现的值
- **输入即搜索（search-as-you-type）**：用户还没输完就给出候选结果
- **复合词语言**：德语/荷兰语等将多个词组合成长词

本页按“查询时方案 → 索引时优化”的顺序展开，并重点强调性能与可控性。

## 查询时方案 1：prefix 前缀查询

查找所有以 `W1` 开头的邮编：

```json
GET /my_index/_search
{
  "query": {
    "prefix": {
      "postcode": "W1"
    }
  }
}
```

`prefix` 是词项级查询，不会分析输入，它做的事情近似于：

1. 扫描倒排词典（有序词列表），找到第一个以 `W1` 开头的词
2. 收集该词对应的文档 ID
3. 向后移动，继续收集所有以 `W1` 开头的词，直到遇到不再匹配的词

**重要的性能结论**（权威指南强调）：前缀越短，扫描的词越多；当唯一词很多时，前缀查询的伸缩性并不好。尽量使用更长的前缀，或者转向索引时优化方案（见下文）。

## 查询时方案 2：wildcard / regexp（更灵活，也更危险）

通配符查询（`?` 匹配一个字符，`*` 匹配 0 或多个字符）：

```json
GET /my_index/_search
{
  "query": {
    "wildcard": {
      "postcode": "W?F*HW"
    }
  }
}
```

正则表达式查询：

```json
GET /my_index/_search
{
  "query": {
    "regexp": {
      "postcode": "W[0-9].+"
    }
  }
}
```

它们与 `prefix` 的本质一致：仍然需要扫描词典找出“匹配的词”，再收集对应文档。也因此有相同的风险：

- 避免**左通配**（如 `*foo` 或正则 `.*foo`）：会导致海量扩展
- 对唯一词非常多的字段慎用：会消耗大量资源

### 在 text 字段上使用时的一个常见坑

`prefix`/`wildcard`/`regexp` 都是词项级查询：如果你对 `text` 字段使用它们，它们匹配的是字段里被分析出的**单个词项**，不是“整段文本”。

例如文本 `Quick brown fox` 通常会被分析为 `quick`、`brown`、`fox`：

- 会匹配：`{ "regexp": { "title": "br.*" } }`
- 不会匹配：`{ "regexp": { "title": "Qu.*" } }`（因为索引里是 `quick`）
- 不会匹配：`{ "regexp": { "title": "quick br*" } }`（因为 `quick` 与 `brown` 是两个词项）

## 查询时输入即搜索：match_phrase_prefix

权威指南给出的“最简单的输入即搜索”方式是在查询时用 `match_phrase_prefix`：它和 `match_phrase` 一样做短语约束，但把**最后一个词**当作前缀：

```json
GET /_search
{
  "query": {
    "match_phrase_prefix": {
      "brand": {
        "query": "johnnie walker bl",
        "max_expansions": 50
      }
    }
  }
}
```

要点：

- 只有最后一个词做前缀
- `max_expansions` 很关键：限制前缀能扩展成多少个候选词项
- 用户每输入一个字符就会再触发一次查询，所以必须控制成本

## 索引时优化：把查询成本“搬到写入端”

查询时方案的共同特点是：灵活，但可能慢。权威指南的建议是：在需要低延迟体验（例如 Web 实时搜索）时，把部分匹配的成本转移到索引阶段——只付一次写入代价，换取每次查询更快。

### 1）edge n-grams：索引时输入即搜索

边界 n-gram（edge n-gram）会为词生成前缀序列，例如 `quick`：

- `q`、`qu`、`qui`、`quic`、`quick`

配置一个用于索引的 `edge_ngram` 过滤器 + 分析器（示意）：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [ "lowercase", "autocomplete_filter" ]
        }
      }
    }
  }
}
```

**关键点（来自权威指南）**：不要让搜索端也用 `autocomplete` 分析器，否则查询会被扩展成大量前缀词项（例如 `b br bro ...`），会把不相关结果拉进来。

更合理的做法是：

- 索引时：用 `autocomplete` 生成前缀
- 搜索时：用 `standard`（只保留用户输入的完整词项）

可以在映射里为字段设置 `search_analyzer`（示意）：

```json
PUT /my_index/_mapping
{
  "properties": {
    "name": {
      "type": "text",
      "analyzer": "autocomplete",
      "search_analyzer": "standard"
    }
  }
}
```

### 2）Completion suggester：更快的前缀建议

权威指南指出：对“名字/品牌”这种词序相对固定的补全场景，completion suggester 会把所有可能完成项编码进内存结构（FST），前缀查找速度非常快。

这类能力在本书对应到“建议（Suggester）”体系，详见：

- [建议](./suggestions.md)

### 3）用 edge n-grams 做结构化值（邮编）

邮编这类值通常不需要分词，但仍可用 `keyword` tokenizer 把它作为一个整体 token，再用 edge n-grams 生成前缀（权威指南的技巧）：

- 索引时：`keyword` tokenizer + `edge_ngram`
- 搜索时：`keyword` tokenizer（保持精确）

## 复合词：用 n-grams 做“词内匹配”

对德语等复合词语言，权威指南给出另一种更通用但更“霰弹枪”的做法：对词做 n-gram（常从 trigram 开始），把词内部片段也索引出来。

示意（trigram）：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "trigrams_filter": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 3
        }
      },
      "analyzer": {
        "trigrams": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [ "lowercase", "trigrams_filter" ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "trigrams"
      }
    }
  }
}
```

这种方法能显著提升召回，但可能引入奇怪匹配（例如只命中一个 trigram 的噪声）。权威指南建议用 `minimum_should_match` 把噪声剪掉。

## 小结

- `prefix`/`wildcard`/`regexp` 都会扫描词典扩展候选词项：越短越危险，左通配最危险
- `match_phrase_prefix` 可快速实现“查询时输入即搜索”，务必设置 `max_expansions`
- 低延迟场景建议做索引时优化：edge n-grams + `search_analyzer` 分离
- 复合词可用 n-grams 做词内匹配，但要配合 `minimum_should_match` 控噪

下一步可以继续阅读：

- [邻近匹配](./proximity-matching.md)
- [多字段搜索](./multi-field-search.md)
- [建议](./suggestions.md)

## 参考手册（API 与参数）

- [前缀查询与 Prefix 相关选项（功能手册）]({{< relref "docs/features/query-dsl/term-based-query/prefix.md" >}})
- [通配符查询（功能手册）]({{< relref "docs/features/query-dsl/term-based-query/wildcard.md" >}})
- [正则查询与语法（功能手册）]({{< relref "docs/features/query-dsl/term-based-query/regexp.md" >}})

