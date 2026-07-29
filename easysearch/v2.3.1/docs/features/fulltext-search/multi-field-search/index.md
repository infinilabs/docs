---
title: "多字段搜索"
date: 0001-01-01
description: "在多个字段上组织相关性：bool、dis_max、multi_match（best_fields/most_fields/cross_fields）与 copy_to。"
summary: "多字段搜索 #  真实的搜索很少是单个字段的 match。更常见的是：
 同一条查询要在多个字段上执行（标题/正文/标签） 不同的查询片段要映射到不同字段（标题 vs 作者） 多个字段共同组成一个“实体”（姓名、地址），每个词可能落在不同字段里  难点在于：匹配结果怎么合并、相关性怎么计算。这一页会按权威指南的三种典型场景，给出可直接落地的查询结构。
场景 1：多字符串、多字段（你知道每个词该搜哪个字段） #  例如你要找作者 Leo Tolstoy 写的《War and Peace》：
GET /_search { &#34;query&#34;: { &#34;bool&#34;: { &#34;should&#34;: [ { &#34;match&#34;: { &#34;title&#34;: &#34;War and Peace&#34; } }, { &#34;match&#34;: { &#34;author&#34;: &#34;Leo Tolstoy&#34; } } ] } } } bool 查询采用“匹配越多越好”的策略：匹配到更多 should 子句的文档会得到更高 _score。
语句优先级：用 boost 做权重分配 #  当有些子句更重要（例如标题、作者比译者更重要），可以给关键子句加 boost：
GET /_search { &#34;query&#34;: { &#34;bool&#34;: { &#34;should&#34;: [ { &#34;match&#34;: { &#34;title&#34;: { &#34;query&#34;: &#34;War and Peace&#34;, &#34;boost&#34;: 2 } } }, { &#34;match&#34;: { &#34;author&#34;: { &#34;query&#34;: &#34;Leo Tolstoy&#34;, &#34;boost&#34;: 2 } } }, { &#34;bool&#34;: { &#34;should&#34;: [ { &#34;match&#34;: { &#34;translator&#34;: &#34;Constance Garnett&#34; } }, { &#34;match&#34;: { &#34;translator&#34;: &#34;Louise Maude&#34; } } ] } } ] } } }  经验：boost 往往不需要非常大。通常从 1~10 试起，配合真实样本不断调试更可靠。"
---


# 多字段搜索

真实的搜索很少是单个字段的 `match`。更常见的是：

- 同一条查询要在多个字段上执行（标题/正文/标签）
- 不同的查询片段要映射到不同字段（标题 vs 作者）
- 多个字段共同组成一个“实体”（姓名、地址），每个词可能落在不同字段里

难点在于：**匹配结果怎么合并、相关性怎么计算**。这一页会按权威指南的三种典型场景，给出可直接落地的查询结构。

## 场景 1：多字符串、多字段（你知道每个词该搜哪个字段）

例如你要找作者 Leo Tolstoy 写的《War and Peace》：

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" } },
        { "match": { "author": "Leo Tolstoy" } }
      ]
    }
  }
}
```

`bool` 查询采用“匹配越多越好”的策略：匹配到更多 `should` 子句的文档会得到更高 `_score`。

### 语句优先级：用 boost 做权重分配

当有些子句更重要（例如标题、作者比译者更重要），可以给关键子句加 `boost`：

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  { "query": "War and Peace", "boost": 2 } } },
        { "match": { "author": { "query": "Leo Tolstoy",   "boost": 2 } } },
        {
          "bool": {
            "should": [
              { "match": { "translator": "Constance Garnett" } },
              { "match": { "translator": "Louise Maude" } }
            ]
          }
        }
      ]
    }
  }
}
```

> 经验：`boost` 往往不需要非常大。通常从 `1~10` 试起，配合真实样本不断调试更可靠。

## 场景 2：单字符串、多字段（字段彼此“竞争”）：best_fields

当用户输入的是一个“概念/短语”，并且你希望它尽量在**同一个字段**里被完整命中（例如：标题或正文之一），这就是 **best_fields** 场景。

一个典型反例：用 `bool/should` 直接把 `title` 与 `body` 的 `match` 叠加，可能会把“两个字段都命中一个词”的文档排到“单个字段命中两个词”的文档前面。

解决办法是用 `dis_max`：返回所有匹配，但把**最佳匹配字段**的得分作为主得分。

### dis_max 与 tie_breaker

纯 `dis_max` 只吃“最佳字段”的分数，可能忽略“另一个字段也命中”的价值。可以用 `tie_breaker` 做折中：

- **主分数** = 最佳匹配字段的 `_score`
- **附加分** = 其他匹配字段 `_score * tie_breaker`

### multi_match：best_fields 的更简洁写法

`multi_match` 会为每个字段生成 `match`，并按 `type` 选择合并策略。默认类型就是 `best_fields`（本质上就是 `dis_max`）。

```json
GET /_search
{
  "query": {
    "multi_match": {
      "query": "quick brown fox",
      "type": "best_fields",
      "fields": [ "title", "body" ],
      "tie_breaker": 0.3,
      "minimum_should_match": "30%"
    }
  }
}
```

### 字段名通配与单字段 boost

`fields` 支持通配写法与 `^` 权重语法：

```json
{
  "query": {
    "multi_match": {
      "query": "quick brown fox",
      "fields": [ "*_title", "chapter_title^2" ]
    }
  }
}
```

## 场景 3：同一段文本用多种方式索引（“信号叠加”）：most_fields

为了提高召回与排序质量，一个常见策略是：**把同一段文本索引到多个字段**，每个字段使用不同分析链：

- 一个字段更“宽松”（词干、同义词、去音标等）→ 扩大召回
- 另一些字段更“精确”（原词、shingles 等）→ 提供额外相关性信号

这就是 **most_fields**：匹配字段越多越好，把各字段的得分合并起来。

```json
GET /_search
{
  "query": {
    "multi_match": {
      "query": "jumping rabbits",
      "type": "most_fields",
      "fields": [ "title", "title.std" ]
    }
  }
}
```

你可以为“主字段”加大权重，让宽松字段主导召回，而精确字段更多起到“加分信号”的作用：

```json
{
  "query": {
    "multi_match": {
      "query": "jumping rabbits",
      "type": "most_fields",
      "fields": [ "title^10", "title.std" ]
    }
  }
}
```

> `most_fields` 适合“同一内容、多种分析链”的场景；它并不适合“实体由多个字段拼出来”的场景（见下一节）。

## 跨字段实体搜索：cross_fields（词中心式）

对于 `person`、`address` 这类实体，用户输入 `Peter Smith`，词可能分别落在 `first_name` 与 `last_name` 字段中。此时用 `best_fields`（找一个最佳字段）显然不对；用 `most_fields`（字段中心式）也会带来三个问题（权威指南的总结）：

- 在多个字段重复命中同一个词，可能比在不同字段命中更多不同词更“高分”
- `operator` / `minimum_should_match` 很难按预期工作（会要求同一个字段里包含所有词）
- TF/IDF 在不同字段上独立统计，可能导致“名/姓”的常见度被错误放大

`cross_fields` 使用**词中心式**逻辑：把多个字段视为一个大字段，对每个词在所有字段里找匹配，并对 IDF 做“混合/融合”（blended）处理。

```json
GET /_search
{
  "query": {
    "multi_match": {
      "query": "peter smith",
      "type": "cross_fields",
      "operator": "and",
      "fields": [ "first_name", "last_name" ]
    }
  }
}
```

### cross_fields 的关键前提：分析器要一致

为了让 `cross_fields` 最优工作，被合并的字段应使用**相同的分析器**。分析器不同的字段会被分组，并以类似 `best_fields` 的方式加入结果中，这会影响 `operator`/`minimum_should_match` 的语义。

### 跨字段也可以按字段 boost

```json
{
  "query": {
    "multi_match": {
      "query": "peter smith",
      "type": "cross_fields",
      "fields": [ "title^2", "description" ]
    }
  }
}
```

## 索引时方案：copy_to（自定义“全字段”）

如果你希望在索引时就把多个字段拼成一个“可搜索的组合字段”，可以用 `copy_to`（权威指南称为自定义 `_all`）：

```json
PUT /my_index
{
  "mappings": {
    "properties": {
      "first_name": { "type": "text", "copy_to": "full_name" },
      "last_name":  { "type": "text", "copy_to": "full_name" },
      "full_name":  { "type": "text" }
    }
  }
}
```

这样你既可以分别搜 `first_name` / `last_name`，也可以用 `full_name` 一把梭做实体搜索。

## 不要把“精确值字段”混进 multi_match 的全文字段里

权威指南强调：把“全文字段”和“精确值字段”混在同一个 `multi_match` 里通常没意义。

原因是：精确值字段（例如 `keyword`）不会分词，查询 `peter smith` 会被当作一个整体去匹配 `title:"peter smith"`，几乎必然匹配不到。

实践建议：

- 全文相关：用 `text` 字段（`match` / `multi_match`）
- 精确过滤/聚合/排序：用 `keyword` 字段（`term/terms`、聚合、排序）
- 若必须混用，通常应拆成 `bool`：全文部分做 `must/should`，精确条件做 `filter`

## 小结：怎么选？

- **已知“词 → 字段”映射**：用 `bool` 组织多条 `match`，用 `boost` 控权重
- **字段竞争（title vs body）**：用 `best_fields`（`dis_max` / `tie_breaker`）
- **同一内容多分析链（信号叠加）**：用 `most_fields`（multi-fields）
- **实体跨字段（名/姓、地址分段）**：优先 `cross_fields` 或索引时用 `copy_to` 组合字段

下一步可以继续阅读：

- [全文检索](./fulltext-search.md)
- [邻近匹配](./proximity-matching.md)
- [Mapping 模式与最佳实践](../../best-practices/data-modeling/mapping-patterns.md)

## 参考手册（API 与参数）

- [multi_match 查询（功能手册）]({{< relref "/docs/features/fulltext-search/full-text/multi-match.md" >}})
- [DisMax / dis_max 查询（功能手册）]({{< relref "docs/features/query-dsl/compound-query/disjunction-max.md" >}})
- [bool 查询（功能手册）]({{< relref "docs/features/query-dsl/compound-query/bool.md" >}})

