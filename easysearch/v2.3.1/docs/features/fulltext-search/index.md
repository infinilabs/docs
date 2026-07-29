---
title: "全文搜索"
date: 0001-01-01
description: "全文检索、多字段搜索、邻近匹配、高亮、建议、相关性调优。"
summary: "全文搜索是 Easysearch 最常用的搜索场景。本章节涵盖从基础教程到高级调优的全文搜索能力，包括分词、相关性评分、多字段搜索等。
快速入门 #  从最基础的全文查询开始：
  全文检索基础：BM25 算法、分词、匹配机制  多字段搜索：best_fields、most_fields、cross_fields 策略  邻近匹配：短语匹配、slop 参数、多值字段  部分匹配：前缀、通配符、正则、模糊查询  高亮：标记匹配文本片段  自动建议：自动补全、拼写纠正  全文查询类型 #  所有全文查询都支持分词与相关性计分。详细用法见 全文查询 章节：
   查询 说明 常见用途      match 基础全文查询，单字段 简单关键词搜索    match_phrase 短语匹配 短语搜索    match_phrase_prefix 短语前缀 搜索框自动补全    match_bool_prefix 布尔前缀匹配 跨字段自动补全    multi_match 多字段全文查询 在多个字段中搜索    query_string 查询字符串（支持布尔操作符） 高级用户输入    simple_query_string 简化查询字符串 用户输入容错    intervals 高级文本匹配 精细文本匹配控制    Span 查询 #  Span 查询提供了文本级的精细控制，常用于高级文本分析与法律文档检索。详见 查询 DSL &gt; Span 查询 章节。"
---


全文搜索是 Easysearch 最常用的搜索场景。本章节涵盖从基础教程到高级调优的全文搜索能力，包括分词、相关性评分、多字段搜索等。

## 快速入门

从最基础的全文查询开始：

1. [全文检索基础](./fulltext-search.md)：BM25 算法、分词、匹配机制
2. [多字段搜索](./multi-field-search.md)：best_fields、most_fields、cross_fields 策略
3. [邻近匹配](./proximity-matching.md)：短语匹配、slop 参数、多值字段
4. [部分匹配](./partial-matching.md)：前缀、通配符、正则、模糊查询
5. [高亮](./highlighting.md)：标记匹配文本片段
6. [自动建议](./suggestions.md)：自动补全、拼写纠正

## 全文查询类型

所有全文查询都支持分词与相关性计分。详细用法见 [全文查询](./full-text/) 章节：

| 查询 | 说明 | 常见用途 |
|------|------|--------|
| [match](./full-text/match.md) | 基础全文查询，单字段 | 简单关键词搜索 |
| [match_phrase](./full-text/match-phrase.md) | 短语匹配 | 短语搜索 |
| [match_phrase_prefix](./full-text/match-phrase-prefix.md) | 短语前缀 | 搜索框自动补全 |
| [match_bool_prefix](./full-text/match-bool-prefix.md) | 布尔前缀匹配 | 跨字段自动补全 |
| [multi_match](./full-text/multi-match.md) | 多字段全文查询 | 在多个字段中搜索 |
| [query_string](./full-text/query-string.md) | 查询字符串（支持布尔操作符） | 高级用户输入 |
| [simple_query_string](./full-text/simple-query-string.md) | 简化查询字符串 | 用户输入容错 |
| [intervals](./full-text/intervals.md) | 高级文本匹配 | 精细文本匹配控制 |

## Span 查询

Span 查询提供了文本级的精细控制，常用于高级文本分析与法律文档检索。详见 [查询 DSL > Span 查询]({{< relref "/docs/features/query-dsl/span/_index.md" >}}) 章节。

## 分页与排序

| 功能 | 说明 |
|------|------|
| [分页与排序](./pagination-and-sorting.md) | from/size、search_after、scroll、按字段排序、按相关性排序 |

## 相关性调优

影响搜索结果顺序的各种因素与调优方法：

| 主题 | 说明 |
|------|------|
| [相关性基础](./relevance/scoring-basics.md) | TF、IDF、BM25 算法基础 |
| [Boosting 与评分](./relevance/boosting.md) | query-time boosting、index-time boosting |
| [调试与解释](./relevance/debug-and-explain.md) | explain API、_score 调试 |
| [相关性配方](./relevance/relevance-recipes.md) | 常见相关性问题的解决方案 |

---

## 常见用途

| 需求 | 实现方式 |
|------|---------|
| 简单关键词搜索 | [match](./full-text/match.md) 查询 |
| 短语搜索（"完整的短语"） | [match_phrase](./full-text/match-phrase.md) |
| 搜索框自动补全 | [match_phrase_prefix](./full-text/match-phrase-prefix.md) + 前缀索引 |
| 多字段搜索（标题 + 内容） | [multi_match](./full-text/multi-match.md) with best_fields |
| 中文分词全文搜索 | match 查询 + IK 分词器 |
| 相关性调优 | [相关性](./relevance/) 章节 |
| 搜索结果高亮 | [高亮](./highlighting.md) 参数 |

## 最佳实践

- [查询调优与慢查询排查]({{< relref "/docs/best-practices/query-tuning.md" >}})：查询性能优化、慢查询诊断
- [多字段搜索的数据建模]({{< relref "/docs/best-practices/data-modeling/document-design.md" >}})：multi-fields 字段设计
