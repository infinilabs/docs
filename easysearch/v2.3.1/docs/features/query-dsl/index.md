---
title: "查询 DSL"
date: 0001-01-01
description: "Query DSL 基础、参数说明、查询类型参考（精确、全文、复合、关联、Span、Geo、专业）、搜索结果处理。"
summary: "本章节是 Easysearch 查询语言（Query DSL）的完整参考。无论你是想了解查询的工作原理，还是需要查阅具体的查询语法，都可以在这里找到答案。
基础教程 #  从基本概念开始，逐步深入：
  Query DSL 基础：查询结构、查询上下文、请求体搜索  结构化搜索：精确值查询、范围查询、存在性查询  查询与过滤上下文：理解计分与性能   查询类型参考 #  Easysearch 支持 50+ 种查询类型，按用途分为以下几大类：
精确查询（Term-Based Query） #  用于精确值匹配，不进行分词。详见 精确查询。
   查询 说明      term 精确匹配单个值    terms 精确匹配多个值（支持条件查找）    range 范围查询（数值、日期、字符串）    prefix 前缀匹配    wildcard 通配符查询（*、?）    regexp 正则表达式（Lucene 语法）    fuzzy 模糊匹配（编辑距离）    exists 存在性检查    ids 按文档 ID 查询    terms_set 动态词项匹配（最少匹配数）    全文查询（Full-Text Query） #  支持分词与相关性计分的查询。详见 全文搜索。"
---


本章节是 Easysearch 查询语言（Query DSL）的完整参考。无论你是想了解查询的工作原理，还是需要查阅具体的查询语法，都可以在这里找到答案。

## 基础教程

从基本概念开始，逐步深入：

1. [Query DSL 基础](./query-dsl-basics.md)：查询结构、查询上下文、请求体搜索
2. [结构化搜索](./structured-search.md)：精确值查询、范围查询、存在性查询
3. [查询与过滤上下文](./query-filter-context.md)：理解计分与性能

---

## 查询类型参考

Easysearch 支持 **50+ 种查询类型**，按用途分为以下几大类：

### 精确查询（Term-Based Query）

用于精确值匹配，不进行分词。详见 [精确查询](./term-based-query/)。

| 查询 | 说明 |
|------|------|
| [term](./term-based-query/term.md) | 精确匹配单个值 |
| [terms](./term-based-query/terms.md) | 精确匹配多个值（支持条件查找） |
| [range](./term-based-query/range.md) | 范围查询（数值、日期、字符串） |
| [prefix](./term-based-query/prefix.md) | 前缀匹配 |
| [wildcard](./term-based-query/wildcard.md) | 通配符查询（`*`、`?`） |
| [regexp](./term-based-query/regexp.md) | 正则表达式（Lucene 语法） |
| [fuzzy](./term-based-query/fuzzy.md) | 模糊匹配（编辑距离） |
| [exists](./term-based-query/exists.md) | 存在性检查 |
| [ids](./term-based-query/ids.md) | 按文档 ID 查询 |
| [terms_set](./term-based-query/terms-set.md) | 动态词项匹配（最少匹配数） |

### 全文查询（Full-Text Query）

支持分词与相关性计分的查询。详见 [全文搜索]({{< relref "/docs/features/fulltext-search/_index.md" >}})。

| 查询 | 说明 |
|------|------|
| [match]({{< relref "/docs/features/fulltext-search/full-text/match.md" >}}) | 标准全文查询 |
| [match_phrase]({{< relref "/docs/features/fulltext-search/full-text/match-phrase.md" >}}) | 短语匹配（词序+位置） |
| [match_phrase_prefix]({{< relref "/docs/features/fulltext-search/full-text/match-phrase-prefix.md" >}}) | 短语前缀匹配 |
| [match_bool_prefix]({{< relref "/docs/features/fulltext-search/full-text/match-bool-prefix.md" >}}) | 布尔前缀匹配 |
| [multi_match]({{< relref "/docs/features/fulltext-search/full-text/multi-match.md" >}}) | 多字段全文搜索 |
| [query_string]({{< relref "/docs/features/fulltext-search/full-text/query-string.md" >}}) | Lucene 查询字符串语法 |
| [simple_query_string]({{< relref "/docs/features/fulltext-search/full-text/simple-query-string.md" >}}) | 简化查询字符串（容错） |
| [intervals]({{< relref "/docs/features/fulltext-search/full-text/intervals.md" >}}) | 词项间隔匹配 |

### 复合查询（Compound Query）

用于组合多个查询条件。详见 [复合查询](./compound-query/)。

| 查询 | 说明 |
|------|------|
| [bool](./compound-query/bool.md) | 布尔组合（must、should、must_not、filter） |
| [boosting](./compound-query/boosting.md) | 降低特定结果的相关性 |
| [constant_score](./compound-query/constant-score.md) | 常数评分（包装 filter） |
| [dis_max](./compound-query/disjunction-max.md) | 取最高分（多字段） |
| [function_score](./compound-query/function-score.md) | 函数式评分（衰减、随机、脚本） |

### Span 查询

文本级精细控制查询（词项位置、距离、顺序）。详见 [Span 查询](./span/)。

| 查询 | 说明 |
|------|------|
| [span_term](./span/span-term.md) | 单个词项的位置查询 |
| [span_multi](./span/span-multi-term.md) | 多词项 span（包装 prefix/wildcard/regexp） |
| [span_near](./span/span-near.md) | 词项距离约束（slop） |
| [span_or](./span/span-or.md) | 多个 span 的 OR |
| [span_not](./span/span-not.md) | 排除特定位置 |
| [span_first](./span/span-first.md) | 限制匹配在前 N 个位置内 |
| [span_containing](./span/span-containing.md) | 包含另一个 span 的匹配 |
| [span_within](./span/span-within.md) | 被另一个 span 包含的匹配 |
| [span_field_masking](./span/span-field-masking.md) | 跨字段 span 查询 |

### 关联查询（Joining Query）

跨文档的关联查询。详见 [关联查询](./joining/)。

| 查询 | 说明 |
|------|------|
| [nested](./joining/nested.md) | 嵌套文档查询 |
| [has_child](./joining/has-child.md) | 父文档查询（通过子文档条件） |
| [has_parent](./joining/has-parent.md) | 子文档查询（通过父文档条件） |
| [parent_id](./joining/parent-id.md) | 按父文档 ID 查询子文档 |

### 地理查询（Geo Query）

基于地理位置的查询。详见 [地理搜索]({{< relref "/docs/features/geo-search/_index.md" >}})。

| 查询 | 说明 |
|------|------|
| [geo_bounding_box]({{< relref "/docs/features/geo-search/geo-bounding-box.md" >}}) | 矩形区域内的点 |
| [geo_distance]({{< relref "/docs/features/geo-search/geo-distance.md" >}}) | 指定距离内的点 |
| [geo_polygon]({{< relref "/docs/features/geo-search/geo-polygon.md" >}}) | 多边形区域内的点 |
| [geo_shape]({{< relref "/docs/features/geo-search/geo-shape-query.md" >}}) | 与指定形状相交/包含/相离 |

### 专业查询（Specialized Query）

特殊场景的高级查询。详见 [专业查询](./specialized/)。

| 查询 | 说明 |
|------|------|
| [more_like_this](./specialized/more-like-this.md) | 相似文档查询 |
| [percolate](./specialized/percolate.md) | 反向查询（文档匹配规则） |
| [rank_feature](./specialized/rank-feature.md) | 排名特性查询 |
| [script_score](./specialized/script-score.md) | 脚本评分 |
| [distance_feature](./specialized/distance-feature.md) | 距离特性 |
| [script](./specialized/script.md) | 脚本查询（底层接口） |
| [wrapper](./specialized/wrapper.md) | 包装查询（JSON 字符串） |

---

## 搜索结果处理

### 分页、排序与高亮

| 功能 | 说明 |
|------|------|
| [分页与滚动](./paginate-and-scroll.md) | from/size、Scroll、search_after、PIT |
| [排序](./sort.md) | 字段排序、地理距离排序、脚本排序 |
| [高亮](./highlight.md) | 高亮命中词（unified/plain/fvh 三种高亮器） |
| [自动补全](./autocomplete.md) | match_phrase_prefix、Edge N-gram、Completion Suggester |
| [Point In Time (PIT)](./pit-api.md) | 冻结快照式分页搜索 |

### 查询参数与工具

| 参数 | 说明 | 示例 |
|------|------|------|
| [minimum_should_match](./minimum-should-match.md) | bool 查询中 should 的最少匹配数 | `minimum_should_match: 2` |
| [rewrite](./rewrite-parameter.md) | 多词项查询的重写策略 | `rewrite: constant_score` |
| [正则表达式](./regex-syntax.md) | Lucene 正则引擎支持的语法 | |
| [Match All / Match None](./match-all.md) | 匹配所有/不匹配任何文档 | |

### 高级功能

| 功能 | 说明 |
|------|------|
| [Search Template](./search-template.md) | Mustache 模板化查询，支持参数化 |
| [搜索管道](./search-pipelines/) | 查询改写、结果重排、后处理 |
| [结果折叠](./collapse.md) | 按字段值去重折叠（field collapsing） |
| [查询重打分](./rescore.md) | 对 Top-N 结果用更精细的查询重新打分 |

---

## 快速导航

| 场景 | 推荐查询 |
|------|---------|
| 查找所有数据 | [match_all](./match-all.md) |
| 精确查找一个值 | [term](./term-based-query/term.md) |
| 精确查找多个值 | [terms](./term-based-query/terms.md) |
| 范围查询（日期、数值） | [range](./term-based-query/range.md) |
| 全文搜索关键词 | [match]({{< relref "/docs/features/fulltext-search/full-text/match.md" >}}) |
| 多字段全文搜索 | [multi_match]({{< relref "/docs/features/fulltext-search/full-text/multi-match.md" >}}) |
| 短语搜索 | [match_phrase]({{< relref "/docs/features/fulltext-search/full-text/match-phrase.md" >}}) |
| 布尔组合（AND/OR/NOT） | [bool](./compound-query/bool.md) |
| 嵌套文档查询 | [nested](./joining/nested.md) |
| 相似文档查询 | [more_like_this](./specialized/more-like-this.md) |
| 地理距离搜索 | [geo_distance]({{< relref "/docs/features/geo-search/geo-distance.md" >}}) |
| 搜索建议/自动补全 | [自动补全](./autocomplete.md) |
