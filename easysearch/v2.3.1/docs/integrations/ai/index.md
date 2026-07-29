---
title: "AI 集成"
date: 0001-01-01
description: "对接 Embedding 服务、向量检索与 LLM/RAG 系统，让 Easysearch 成为 AI 工作流中的检索中枢。"
summary: "本小节关注“Easysearch 在 AI 系统里的位置”，包括：
 如何对接外部 Embedding/模型服务 如何把向量写入 Easysearch 并做向量/语义检索 如何在 RAG/问答系统中，以 Easysearch 为检索层构建整体链路  子页面：
  Embedding 服务接入：与文本/多模态 Embedding 服务的对接模式  向量工作流与 Hybrid 检索：向量写入、批量导入与 BM25 + 向量的组合  RAG 与 LLM 集成：以 Easysearch 为检索底座的问答/助手类系统  最佳实践 #    AI 搜索与向量检索架构：多路召回策略、Hybrid 搜索、LLM 集成  向量字段建模：向量维度选择、写入策略、存储优化  NLP 自然语言处理：分词、同义词、向量化基础  "
---


本小节关注“Easysearch 在 AI 系统里的位置”，包括：

- 如何对接外部 Embedding/模型服务
- 如何把向量写入 Easysearch 并做向量/语义检索
- 如何在 RAG/问答系统中，以 Easysearch 为检索层构建整体链路

子页面：

- [Embedding 服务接入]({{< relref "./embedding-service" >}})：与文本/多模态 Embedding 服务的对接模式
- [向量工作流与 Hybrid 检索]({{< relref "./vector-workflow" >}})：向量写入、批量导入与 BM25 + 向量的组合
- [RAG 与 LLM 集成]({{< relref "./rag-and-llm" >}})：以 Easysearch 为检索底座的问答/助手类系统

## 最佳实践

- [AI 搜索与向量检索架构]({{< relref "/docs/best-practices/ai-search-architecture.md" >}})：多路召回策略、Hybrid 搜索、LLM 集成
- [向量字段建模]({{< relref "/docs/best-practices/data-modeling/vector-fields.md" >}})：向量维度选择、写入策略、存储优化
- [NLP 自然语言处理]({{< relref "/docs/fundamentals/nlp.md" >}})：分词、同义词、向量化基础
