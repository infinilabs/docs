---
title: "英语形态分词过滤器（English Morphology）"
date: 0001-01-01
summary: "English Morphology 分词过滤器 #  english_morphology 分词过滤器是基于 Lucene Morphology 库的语言处理组件。与传统的算法分词器不同，它不依赖于简单的字符裁剪规则，而是通过语言学词典对单词进行深度的词形还原（Lemmatization）。
相关指南（先读这些） #    文本分析：词干提取  文本分析：规范化  核心处理逻辑 #  该过滤器在处理文本时遵循“字典比对 + 语义还原”的原则，其核心逻辑包括：
 屈折变化还原 (Inflectional)：处理动词时态（went → go）、名词单复数（children → child）等。 派生词识别 (Derivational)：识别单词间的词根关联，如将“执行者名词”关联至“动作动词”（runner → run）。 多路径索引 (Token Expansion)：当一个单词具有多重身份时，同时保留原词和还原后的原型（如 running → running, run）。  安装与使用 #  详见 英语形态分词器 部分。"
---


# English Morphology 分词过滤器

`english_morphology` 分词过滤器是基于 Lucene Morphology 库的语言处理组件。与传统的算法分词器不同，它不依赖于简单的字符裁剪规则，而是通过语言学词典对单词进行深度的词形还原（Lemmatization）。

## 相关指南（先读这些）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 核心处理逻辑
该过滤器在处理文本时遵循“字典比对 + 语义还原”的原则，其核心逻辑包括：
- 屈折变化还原 (Inflectional)：处理动词时态（went → go）、名词单复数（children → child）等。
- 派生词识别 (Derivational)：识别单词间的词根关联，如将“执行者名词”关联至“动作动词”（runner → run）。
- 多路径索引 (Token Expansion)：当一个单词具有多重身份时，同时保留原词和还原后的原型（如 running → running, run）。

## 安装与使用
详见 [英语形态分词器](../analyzers/english-morphology-analyzer.md) 部分。

