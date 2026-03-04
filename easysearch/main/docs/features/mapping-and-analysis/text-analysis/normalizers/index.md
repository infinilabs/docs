---
title: "规范化器（Normalizers）"
date: 0001-01-01
summary: "规范化器（Normalizers） #  相关指南 #   概念指南 → 理解归一化原理，请阅读 归一化与规范化器
  概述 #  Normalizer 类似于分析器，但有一个关键区别：它只输出单个词元。它没有分词器，只能使用字符级别的过滤器（如大小写转换、ASCII 折叠），不支持分词、同义词或词干提取等操作。
Normalizer 专用于 keyword 字段类型，在索引和查询之前对整个字段值进行字符级归一化。
Normalizer vs Analyzer #     特性 Analyzer Normalizer     输出词元数 多个 1 个   包含分词器 ✅ ❌   适用字段类型 text keyword   支持同义词/词干 ✅ ❌   支持大小写转换 ✅ ✅   支持字符归一化 ✅ ✅     规范化器列表 #     规范化器 说明      lowercase 内置规范化器，将整个值转为小写    自定义规范化器 组合字符过滤器和词元过滤器构建自定义归一化逻辑     兼容的过滤器 #  Normalizer 只接受字符级操作的过滤器，不支持会改变词元数量的过滤器。"
---


# 规范化器（Normalizers）

## 相关指南

> **概念指南** → 理解归一化原理，请阅读 [归一化与规范化器]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization" >}})

---

## 概述

Normalizer 类似于分析器，但有一个关键区别：**它只输出单个词元**。它没有分词器，只能使用字符级别的过滤器（如大小写转换、ASCII 折叠），不支持分词、同义词或词干提取等操作。

Normalizer 专用于 `keyword` 字段类型，在索引和查询之前对整个字段值进行字符级归一化。

### Normalizer vs Analyzer

| 特性 | Analyzer | Normalizer |
|------|----------|------------|
| 输出词元数 | 多个 | **1 个** |
| 包含分词器 | ✅ | ❌ |
| 适用字段类型 | `text` | `keyword` |
| 支持同义词/词干 | ✅ | ❌ |
| 支持大小写转换 | ✅ | ✅ |
| 支持字符归一化 | ✅ | ✅ |

---

## 规范化器列表

| 规范化器 | 说明 |
|----------|------|
| [lowercase]({{< relref "./lowercase" >}}) | 内置规范化器，将整个值转为小写 |
| [自定义规范化器]({{< relref "./custom" >}}) | 组合字符过滤器和词元过滤器构建自定义归一化逻辑 |

---

## 兼容的过滤器

Normalizer 只接受字符级操作的过滤器，不支持会改变词元数量的过滤器。

### 字符过滤器

| 过滤器 | 说明 |
|--------|------|
| [`mapping`]({{< relref "../character-filters/mapping" >}}) | 字符映射替换 |
| [`pattern_replace`]({{< relref "../character-filters/pattern-replace" >}}) | 正则表达式替换 |

### 词元过滤器

| 过滤器 | 说明 |
|--------|------|
| [`lowercase`]({{< relref "../token-filters/lowercase" >}}) | 转小写 |
| [`uppercase`]({{< relref "../token-filters/uppercase" >}}) | 转大写 |
| [`asciifolding`]({{< relref "../token-filters/ascii-folding" >}}) | ASCII 折叠（移除变音符号） |
| [`decimal_digit`]({{< relref "../token-filters/decimal-digit" >}}) | Unicode 数字 → ASCII 数字 |
| [`cjk_width`]({{< relref "../token-filters/cjk-width" >}}) | 全角/半角字符归一化 |
| [`trim`]({{< relref "../token-filters/trim" >}}) | 去除首尾空白 |
| [`elision`]({{< relref "../token-filters/elision" >}}) | 移除省音（如法语 l', d'） |
| [`arabic_normalization`]({{< relref "../token-filters/arabic-normalization" >}}) | 阿拉伯语字符归一化 |
| [`bengali_normalization`]({{< relref "../token-filters/bengali-normalization" >}}) | 孟加拉语字符归一化 |
| [`german_normalization`]({{< relref "../token-filters/german-normalization" >}}) | 德语 ä→a, ß→ss |
| [`hindi_normalization`]({{< relref "../token-filters/hindi-normalization" >}}) | 印地语字符归一化 |
| [`indic_normalization`]({{< relref "../token-filters/indic-normalization" >}}) | 印度语系通用归一化 |
| [`persian_normalization`]({{< relref "../token-filters/persian-normalization" >}}) | 波斯语字符归一化 |
| [`sorani_normalization`]({{< relref "../token-filters/sorani-normalization" >}}) | 索拉尼语字符归一化 |
| [`scandinavian_folding`]({{< relref "../token-filters/scandinavian-folding" >}}) | 斯堪的纳维亚字符折叠 |
| [`scandinavian_normalization`]({{< relref "../token-filters/scandinavian-normalization" >}}) | 斯堪的纳维亚字符归一化 |
| [`serbian_normalization`]({{< relref "../token-filters/serbian-normalization" >}}) | 塞尔维亚语字符归一化 |

---

## 注意事项

1. **对聚合生效**：使用 normalizer 的 keyword 字段在 terms 聚合中返回归一化后的值
2. **对排序生效**：排序时也使用归一化后的值
3. **不可逆**：归一化后原始值仅保存在 `_source` 中，keyword 字段本身存储归一化后的形式
4. **每个字段一个**：每个 keyword 字段只能绑定一个 normalizer
5. **同时影响索引和查询**：normalizer 在索引时和查询时都会执行，确保一致性
