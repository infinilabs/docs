---
title: "复合查询"
date: 0001-01-01
description: "Bool、boosting、constant_score、dis_max、function_score 等复合查询。"
summary: "复合查询用于组合多个查询条件，支持各种逻辑关系（AND、OR、NOT）和评分策略。
查询类型 #     查询 说明 使用场景      bool 布尔组合（must、should、must_not、filter） 最常用的组合查询    boosting 提升评分（positive query + negative boosting） 降低特定结果的相关性    constant_score 常数评分（所有结果同分） 过滤+固定评分    dis_max 取最高分（多字段查询） 多字段搜索，取最佳匹配    function_score 函数式评分（自定义评分函数） 复杂评分逻辑、距离衰减、时间衰减    特点 #   灵活的组合：支持任意嵌套和组合 评分策略多样：支持多种评分计算方式 逻辑清晰：明确的 AND/OR/NOT 语义 性能可控：支持 filter 上下文以提升性能  bool 查询子句说明 #     子句 说明 计分     must 必须匹配（AND） 是   filter 必须匹配（AND） 否，用于过滤   should 应该匹配（OR） 是，可设置最少匹配数   must_not 必须不匹配（NOT） 否    推荐阅读顺序 #    bool：最常用的组合查询  constant_score：过滤+固定评分  dis_max：多字段最高分  boosting：降低特定结果评分  function_score：自定义评分函数  常见用途 #     场景 推荐查询     关键词 AND 分类 AND 日期范围  bool with must   关键词搜索，排除某类结果  bool with must_not   多个搜索词，任一匹配  bool with should   过滤+评分  constant_score   多字段搜索，取最佳字段  dis_max   距离越近评分越高  function_score with gauss decay   新文档评分更高  function_score with linear decay    "
---


复合查询用于组合多个查询条件，支持各种逻辑关系（AND、OR、NOT）和评分策略。

## 查询类型

| 查询 | 说明 | 使用场景 |
|------|------|---------|
| [bool](./bool.md) | 布尔组合（must、should、must_not、filter） | 最常用的组合查询 |
| [boosting](./boosting.md) | 提升评分（positive query + negative boosting） | 降低特定结果的相关性 |
| [constant_score](./constant-score.md) | 常数评分（所有结果同分） | 过滤+固定评分 |
| [dis_max](./disjunction-max.md) | 取最高分（多字段查询） | 多字段搜索，取最佳匹配 |
| [function_score](./function-score.md) | 函数式评分（自定义评分函数） | 复杂评分逻辑、距离衰减、时间衰减 |

## 特点

- **灵活的组合**：支持任意嵌套和组合
- **评分策略多样**：支持多种评分计算方式
- **逻辑清晰**：明确的 AND/OR/NOT 语义
- **性能可控**：支持 filter 上下文以提升性能

## bool 查询子句说明

| 子句 | 说明 | 计分 |
|------|------|------|
| must | 必须匹配（AND） | 是 |
| filter | 必须匹配（AND） | 否，用于过滤 |
| should | 应该匹配（OR） | 是，可设置最少匹配数 |
| must_not | 必须不匹配（NOT） | 否 |

## 推荐阅读顺序

1. [bool](./bool.md)：最常用的组合查询
2. [constant_score](./constant-score.md)：过滤+固定评分
3. [dis_max](./disjunction-max.md)：多字段最高分
4. [boosting](./boosting.md)：降低特定结果评分
5. [function_score](./function-score.md)：自定义评分函数

## 常见用途

| 场景 | 推荐查询 |
|------|---------|
| 关键词 AND 分类 AND 日期范围 | [bool](./bool.md) with must |
| 关键词搜索，排除某类结果 | [bool](./bool.md) with must_not |
| 多个搜索词，任一匹配 | [bool](./bool.md) with should |
| 过滤+评分 | [constant_score](./constant-score.md) |
| 多字段搜索，取最佳字段 | [dis_max](./disjunction-max.md) |
| 距离越近评分越高 | [function_score](./function-score.md) with gauss decay |
| 新文档评分更高 | [function_score](./function-score.md) with linear decay |
