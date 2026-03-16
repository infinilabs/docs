---
title: "RAG 与 LLM 集成"
date: 0001-01-01
description: "以 Easysearch 为检索层，构建检索增强生成（RAG）与问答系统的整体方案。"
summary: "RAG 与 LLM 集成 #  检索增强生成（Retrieval-Augmented Generation，RAG）是一种将搜索引擎与大语言模型（LLM）结合的架构模式。Easysearch 作为高性能检索层，可以为 LLM 提供精准的上下文信息，显著提升生成质量。
相关指南（先读这些） #    Embedding 服务接入  向量工作流与 Hybrid 检索  Hybrid Search API  LangChain 集成  RAG 架构概览 #  典型的 RAG 流程如下：
用户提问 ↓ 1. 查询改写（可选） ↓ 2. 检索阶段：在 Easysearch 中搜索相关文档 - 全文搜索（BM25） - 向量搜索（kNN） - 混合搜索（Hybrid） ↓ 3. 结果裁剪：选取 Top-K 段落，控制 token 数量 ↓ 4. Prompt 组装：将检索结果 + 用户问题拼接为 Prompt ↓ 5. LLM 生成：调用 LLM API 生成答案 ↓ 6."
---


# RAG 与 LLM 集成

检索增强生成（Retrieval-Augmented Generation，RAG）是一种将搜索引擎与大语言模型（LLM）结合的架构模式。Easysearch 作为高性能检索层，可以为 LLM 提供精准的上下文信息，显著提升生成质量。

## 相关指南（先读这些）

- [Embedding 服务接入]({{< relref "./embedding-service.md" >}})
- [向量工作流与 Hybrid 检索]({{< relref "./vector-workflow.md" >}})
- [Hybrid Search API]({{< relref "./ai-api/hybrid-search.md" >}})
- [LangChain 集成]({{< relref "../third-party/langchain.md" >}})

## RAG 架构概览

典型的 RAG 流程如下：

```
用户提问
  ↓
1. 查询改写（可选）
  ↓
2. 检索阶段：在 Easysearch 中搜索相关文档
   - 全文搜索（BM25）
   - 向量搜索（kNN）
   - 混合搜索（Hybrid）
  ↓
3. 结果裁剪：选取 Top-K 段落，控制 token 数量
  ↓
4. Prompt 组装：将检索结果 + 用户问题拼接为 Prompt
  ↓
5. LLM 生成：调用 LLM API 生成答案
  ↓
6. 返回给用户（可附上来源引用）
```

## 检索策略设计

### 选择搜索模式

| 模式     | 优势                         | 劣势                      | 适用场景              |
| -------- | ---------------------------- | ------------------------- | --------------------- |
| BM25     | 精确匹配，无需向量化         | 无法理解语义相似性        | 关键词明确的 FAQ       |
| kNN      | 语义理解，跨表述匹配         | 需要 Embedding 服务       | 语义搜索、知识问答     |
| Hybrid   | 兼顾精确和语义               | 复杂度稍高                | 推荐首选方案           |

### Hybrid 检索示例

```
POST knowledge_base/_search
{
  "size": 5,
  "_source": ["title", "content", "source_url"],
  "query": {
    "bool": {
      "must": [
        {
          "knn_nearest_neighbors": {
            "field": "content_vector",
            "vec": {
              "values": [0.12, -0.34, ...]
            },
            "model": "lsh",
            "similarity": "cosine",
            "candidates": 50
          }
        }
      ],
      "should": [
        {
          "match": {
            "content": {
              "query": "如何配置 Easysearch 集群",
              "boost": 0.3
            }
          }
        }
      ]
    }
  }
}
```

## Prompt 组装策略

将检索结果拼入 Prompt 时，需要注意 LLM 的上下文窗口限制：

```
你是一个技术助手。请根据以下参考资料回答用户的问题。
如果参考资料中没有足够的信息，请说明你不确定。

## 参考资料

【来源 1】{title_1}
{content_1}

【来源 2】{title_2}
{content_2}

## 用户问题
{user_question}

## 回答
```

### Token 预算管理

| LLM              | 上下文窗口      | 建议检索条数 | 每条最大长度   |
| ----------------- | --------------- | ------------ | -------------- |
| GPT-4o            | 128K tokens     | 5~10 条      | 1000~2000 字   |
| Claude 3.5        | 200K tokens     | 5~15 条      | 1000~3000 字   |
| 通义千问          | 8K~128K tokens  | 3~8 条       | 500~1500 字    |
| 本地小模型（7B）  | 4K~8K tokens    | 2~3 条       | 300~500 字     |

## 多轮对话

在多轮问答场景下，需要将历史对话纳入检索和生成流程：

1. **历史感知检索**：将用户最新问题 + 最近几轮对话合并后做检索
2. **对话历史管理**：只保留最近 N 轮对话，避免超出 token 限制
3. **话题切换检测**：当用户转换话题时，清空之前的检索上下文

## 效果优化

| 优化方向         | 具体做法                                                      |
| ---------------- | ------------------------------------------------------------- |
| 数据质量         | 对文档做分段（chunk），每段 200~500 字，保留完整语义            |
| 查询改写         | 用 LLM 对用户问题做扩展/改写，提升检索召回率                   |
| 重排序           | 对检索结果用 Cross-Encoder 重排序，提升 Top-K 精度             |
| 引用追溯         | 返回结果中附上文档来源链接，增强可信度                          |
| 答案验证         | 让 LLM 判断生成答案是否基于检索结果，减少幻觉                   |


