---
title: "全文查询"
date: 0001-01-01
description: "Match、multi_match、match_phrase、query_string 等全文查询类型参数详解。"
summary: "全文查询族是 Easysearch 进行全文搜索的核心。这些查询会对输入文本进行分词，然后与索引中的倒排索引进行匹配，并根据相关性评分（BM25）返回结果。
相关指南（先读这些） #    全文检索基础：BM25、分词、match/match_phrase/multi_match 概述  多字段搜索策略：best_fields/most_fields/cross_fields  邻近匹配与短语查询：slop、position_increment_gap、shingles  查询类型总览 #     查询类型 描述 适用场景      match 最基础的全文查询，单字段，支持运算符与模糊匹配 简单关键词搜索    match_phrase 短语匹配，词项顺序与距离受控 &ldquo;完整短语&quot;搜索    match_phrase_prefix 短语前缀匹配，最后一个词模糊 搜索框自动补全    match_bool_prefix 布尔前缀匹配，词项位置不限，最后一个词项前缀 跨字段自动补全    multi_match 在多个字段中进行全文搜索 同时搜索标题、内容、标签等    query_string 支持布尔操作符的查询语法 高级用户搜索，支持 +、-、AND、OR    simple_query_string 简化版 query_string，容错能力强 用户输入容错，防止查询语法错误    intervals 高级文本匹配，精细控制词项间距与顺序 法律文档、学术文献的精确匹配    特点 #   分词处理：所有全文查询都会对输入文本进行分词（使用字段配置的分析器） 相关性评分：结果按 BM25 算法进行评分排序 字段配置：分词行为由字段的 analyzer 配置决定（如 IK 分词器用于中文） boost 支持：支持字段级和查询级的评分提升  推荐阅读顺序 #    match 查询：从最基础的查询开始  match_phrase 查询：短语搜索  match_phrase_prefix 查询：自动补全  match_bool_prefix 查询：布尔前缀  multi_match 查询：多字段搜索  query_string 查询：高级语法  simple_query_string 查询：容错搜索  intervals 查询：精细控制  "
---


全文查询族是 Easysearch 进行全文搜索的核心。这些查询会对输入文本进行**分词**，然后与索引中的倒排索引进行匹配，并根据相关性评分（BM25）返回结果。

## 相关指南（先读这些）

- [全文检索基础]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})：BM25、分词、match/match_phrase/multi_match 概述
- [多字段搜索策略]({{< relref "/docs/features/fulltext-search/multi-field-search.md" >}})：best_fields/most_fields/cross_fields
- [邻近匹配与短语查询]({{< relref "/docs/features/fulltext-search/proximity-matching.md" >}})：slop、position_increment_gap、shingles

## 查询类型总览

| 查询类型 | 描述 | 适用场景 |
|---------|------|---------|
| [match](./match.md) | 最基础的全文查询，单字段，支持运算符与模糊匹配 | 简单关键词搜索 |
| [match_phrase](./match-phrase.md) | 短语匹配，词项顺序与距离受控 | "完整短语"搜索 |
| [match_phrase_prefix](./match-phrase-prefix.md) | 短语前缀匹配，最后一个词模糊 | 搜索框自动补全 |
| [match_bool_prefix](./match-bool-prefix.md) | 布尔前缀匹配，词项位置不限，最后一个词项前缀 | 跨字段自动补全 |
| [multi_match](./multi-match.md) | 在多个字段中进行全文搜索 | 同时搜索标题、内容、标签等 |
| [query_string](./query-string.md) | 支持布尔操作符的查询语法 | 高级用户搜索，支持 +、-、AND、OR |
| [simple_query_string](./simple-query-string.md) | 简化版 query_string，容错能力强 | 用户输入容错，防止查询语法错误 |
| [intervals](./intervals.md) | 高级文本匹配，精细控制词项间距与顺序 | 法律文档、学术文献的精确匹配 |

## 特点

- **分词处理**：所有全文查询都会对输入文本进行分词（使用字段配置的分析器）
- **相关性评分**：结果按 BM25 算法进行评分排序
- **字段配置**：分词行为由字段的 analyzer 配置决定（如 IK 分词器用于中文）
- **boost 支持**：支持字段级和查询级的评分提升

## 推荐阅读顺序

1. [match 查询](./match.md)：从最基础的查询开始
2. [match_phrase 查询](./match-phrase.md)：短语搜索
3. [match_phrase_prefix 查询](./match-phrase-prefix.md)：自动补全
4. [match_bool_prefix 查询](./match-bool-prefix.md)：布尔前缀
5. [multi_match 查询](./multi-match.md)：多字段搜索
6. [query_string 查询](./query-string.md)：高级语法
7. [simple_query_string 查询](./simple-query-string.md)：容错搜索
8. [intervals 查询](./intervals.md)：精细控制
