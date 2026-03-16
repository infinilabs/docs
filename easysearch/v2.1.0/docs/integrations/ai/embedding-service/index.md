---
title: "Embedding 服务接入"
date: 0001-01-01
description: "如何对接文本/多模态 Embedding 服务，为 Easysearch 提供向量特征。"
summary: "Embedding 服务接入 #  要在 Easysearch 中使用向量检索，首先需要将文本（或其他数据）转换为向量表示。这个过程需要 Embedding 模型服务的支持。
相关指南（先读这些） #    向量检索  Ingest Text Embedding  Search Text Embedding  LangChain 集成  部署模式 #  模式一：写入链路嵌入（推荐） #  在数据写入 Easysearch 时，通过 Ingest Pipeline 自动调用 Embedding 服务：
应用数据 → Easysearch Ingest Pipeline → 调用 Embedding API → 写入向量字段 优势是写入后即可搜索，无需维护外部向量化流程。参见 Ingest Text Embedding。
模式二：查询链路嵌入 #  搜索时实时将查询文本转换为向量，在 Easysearch 中做 kNN 检索：
用户查询 → 调用 Embedding API → 向量化 → Easysearch kNN 搜索 参见 Search Text Embedding。"
---


# Embedding 服务接入

要在 Easysearch 中使用向量检索，首先需要将文本（或其他数据）转换为向量表示。这个过程需要 Embedding 模型服务的支持。

## 相关指南（先读这些）

- [向量检索]({{< relref "../../features/vector-search/_index.md" >}})
- [Ingest Text Embedding]({{< relref "./ai-api/ingest-text-embedding.md" >}})
- [Search Text Embedding]({{< relref "./ai-api/search-text-embedding.md" >}})
- [LangChain 集成]({{< relref "../third-party/langchain.md" >}})

## 部署模式

### 模式一：写入链路嵌入（推荐）

在数据写入 Easysearch 时，通过 Ingest Pipeline 自动调用 Embedding 服务：

```
应用数据 → Easysearch Ingest Pipeline → 调用 Embedding API → 写入向量字段
```

优势是写入后即可搜索，无需维护外部向量化流程。参见 [Ingest Text Embedding]({{< relref "./ai-api/ingest-text-embedding.md" >}})。

### 模式二：查询链路嵌入

搜索时实时将查询文本转换为向量，在 Easysearch 中做 kNN 检索：

```
用户查询 → 调用 Embedding API → 向量化 → Easysearch kNN 搜索
```

参见 [Search Text Embedding]({{< relref "./ai-api/search-text-embedding.md" >}})。

### 模式三：离线批处理

在应用侧完成向量化，再将向量字段直接写入 Easysearch：

```
原始数据 → 批处理脚本 → 调用 Embedding API → 写入 Easysearch（含向量字段）
```

适合大批量历史数据回填或对延迟不敏感的场景。

## 常用 Embedding 服务

| 服务/模型                | 维度     | 语言支持 | 部署方式         |
| ------------------------ | -------- | -------- | ---------------- |
| OpenAI text-embedding-3  | 256~3072 | 多语言   | 云 API           |
| Cohere Embed             | 1024     | 多语言   | 云 API           |
| BGE / M3E（BAAI）        | 768~1024 | 中英文   | 自部署           |
| Sentence-Transformers    | 384~768  | 多语言   | 自部署           |
| 通义千问 Embedding       | 1536     | 中英文   | 云 API           |

## 自部署 Embedding 服务示例

使用 Python FastAPI 部署一个简单的 Embedding 服务：

```python
from fastapi import FastAPI
from sentence_transformers import SentenceTransformer

app = FastAPI()
model = SentenceTransformer("BAAI/bge-base-zh-v1.5")

@app.post("/embed")
async def embed(texts: list[str]):
    embeddings = model.encode(texts).tolist()
    return {"embeddings": embeddings}
```

启动后即可被 Easysearch 的 Ingest Pipeline 或应用程序调用。

## 实践建议

| 要点           | 建议                                                           |
| -------------- | -------------------------------------------------------------- |
| 维度选择       | 768~1024 维是精度和性能的平衡点，过高维度会增加存储和检索成本  |
| 批量处理       | 一次请求多条文本，减少网络往返                                  |
| 缓存策略       | 对相同文本的 Embedding 结果做缓存，避免重复计算                 |
| 超时设置       | Embedding 服务响应时间波动大，设置合理的超时（如 30s）          |
| 模型一致性     | 写入和查询必须使用同一模型，否则向量空间不兼容                  |


