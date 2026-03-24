---
title: "文档建模"
date: 0001-01-01
description: "字段映射与文本分析的实践应用：字段设计、分析器配置与最佳实践。"
tags: ["Mapping", "文本分析", "分析器", "分词器"]
summary: "Mapping 与文本分析是 Easysearch 搜索能力的两大基石。本章为功能手册 - 实践应用部分。
 📚 基础理论优先：请先阅读 Mapping 与文本分析，了解基本概念和工作原理。
 本章内容导航 #  🔧 应用指南 Mapping 设计
 Mapping 基础 - text/keyword 选择、动态映射陷阱、字段配置  文本分析配置
 文本分析 - 语言分析器选择与配置 词汇识别 - 分词器选择与语言处理 归一化与规范化器 - 大小写、变音符号、Normalizer 配置 词干提取 - 词形还原与召回率优化 停用词 - 性能与表达力的权衡 同义词 - 扩展召回与多词同义词处理 模糊匹配 - 拼写容错与编辑距离   📖 API 参考 映射配置参考
 字段类型 - text、keyword、数值、日期、geo、向量等 映射参数 - analyzer、doc_values、index 等 元数据字段 - _source、_id、_routing 等系统字段  文本分析组件参考"
---


Mapping 与文本分析是 Easysearch 搜索能力的两大基石。本章为**功能手册 - 实践应用部分**。

> 📚 **基础理论优先**：请先阅读 [Mapping 与文本分析]({{< relref "/docs/fundamentals/mapping-analysis-intro.md" >}})，了解基本概念和工作原理。

## 本章内容导航

{{< columns >}}

### 🔧 应用指南

**Mapping 设计**
- [Mapping 基础]({{< relref "mapping-basics" >}}) - text/keyword 选择、动态映射陷阱、字段配置

**文本分析配置**
- [文本分析]({{< relref "text-analysis/" >}}) - 语言分析器选择与配置
- [词汇识别]({{< relref "text-analysis-identifying-words" >}}) - 分词器选择与语言处理
- [归一化与规范化器]({{< relref "text-analysis-normalization" >}}) - 大小写、变音符号、Normalizer 配置
- [词干提取]({{< relref "text-analysis-stemming" >}}) - 词形还原与召回率优化
- [停用词]({{< relref "text-analysis-stopwords" >}}) - 性能与表达力的权衡
- [同义词]({{< relref "text-analysis-synonyms" >}}) - 扩展召回与多词同义词处理
- [模糊匹配]({{< relref "text-analysis-fuzzy" >}}) - 拼写容错与编辑距离

<--->

### 📖 API 参考

**映射配置参考**
- [字段类型]({{< relref "field-types" >}}) - text、keyword、数值、日期、geo、向量等
- [映射参数]({{< relref "mapping-parameters" >}}) - analyzer、doc_values、index 等
- [元数据字段]({{< relref "metadata-field" >}}) - _source、_id、_routing 等系统字段

**文本分析组件参考**
- [文本分析 API]({{< relref "text-analysis/" >}}) - 分析器、分词器、过滤器总览
  - [分析器]({{< relref "text-analysis/analyzers" >}}) - Standard、IK、Pinyin 等
  - [分词器]({{< relref "text-analysis/tokenizers" >}}) - 文本拆分为词项
  - [字符过滤器]({{< relref "text-analysis/character-filters" >}}) - 预处理原始文本
  - [分词过滤器]({{< relref "text-analysis/token-filters" >}}) - 词项后处理

{{< /columns >}}

## 推荐学习路径

### 🚀 快速入门

第一次使用 Mapping 与文本分析：

1. **[基础理论]({{< relref "/docs/fundamentals/mapping-analysis-intro.md" >}})** → 理解精确值 vs 全文、分析器三步处理
2. **[Mapping 基础（深入）]({{< relref "mapping-basics" >}})** → text/keyword 选择、动态映射陷阱
3. **[文本分析]({{< relref "text-analysis/" >}})** → 语言分析器选择与配置
4. **[字段类型]({{< relref "field-types" >}})** → 查阅具体参数

### 🔍 搜索质量优化

遇到召回率或相关性问题：

1. **[基础理论]({{< relref "/docs/fundamentals/mapping-analysis-intro.md" >}})** → 理解分词原理
2. **[词汇识别]({{< relref "text-analysis-identifying-words" >}})** → 检查分词是否正确
3. **[词干提取]({{< relref "text-analysis-stemming" >}})** → 扩大召回
4. **[同义词]({{< relref "text-analysis-synonyms" >}})** → 匹配相关表达
5. **[模糊匹配]({{< relref "text-analysis-fuzzy" >}})** → 容忍拼写错误

### 🏗️ 生产索引设计

规划和优化生产环境索引：

1. **[Mapping 模式与最佳实践]({{< relref "mapping-patterns" >}})** → 避免常见陷阱、多字段策略
2. **[映射参数]({{< relref "mapping-parameters" >}})** → 精细控制字段行为（doc_values、fielddata 等）
3. **[向量字段建模]({{< relref "/docs/best-practices/data-modeling/vector-fields.md" >}})** → 语义搜索场景

## 核心概念速查

| 概念 | 说明 | 典型场景 |
|-----|------|---------|
| `text` 类型 | 全文检索字段，索引时分词 | 标题、正文、评论 |
| `keyword` 类型 | 精确值字段，不分词 | ID、状态、标签、枚举 |
| 分析器 | 字符过滤 → 分词 → Token 过滤的组合 | 控制文本如何被处理 |
| 多字段 | 同一数据以多种方式索引 | `title`(text) + `title.keyword`(keyword) |
| 动态映射 | 自动推断新字段类型 | 快速原型，生产环境需显式映射 |
| `_source` | 存储原始 JSON 文档 | 返回文档、reindex 操作 |

## 常见问题

**Q: 为什么搜索 "iPhone" 找不到 "iphone"？**
→ 检查字段是否为 `keyword`（不分词），或分析器是否缺少 `lowercase` 过滤器

**Q: 为什么搜索 "跑步" 找不到 "跑" 的文档？**
→ 检查分词器，中文需要专门分词器（IK、Jieba 等）

**Q: text 和 keyword 应该选哪个？**
→ 需要全文搜索用 `text`；需要精确过滤/排序/聚合用 `keyword`；两者都需要用多字段

**Q: 修改分析器后为什么不生效？**
→ 已索引数据需要重建索引才能应用新的分析器配置
