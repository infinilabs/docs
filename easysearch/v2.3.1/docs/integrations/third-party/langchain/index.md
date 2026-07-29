---
title: "LangChain 集成"
date: 0001-01-01
description: "LangChain 与 Easysearch 的 RAG 集成。"
summary: "LangChain 集成 #   LangChain 是最流行的 LLM 应用开发框架。通过将 Easysearch 作为 Vector Store，可以构建 RAG（Retrieval-Augmented Generation）应用，让大模型基于企业知识库进行问答。
架构概览 #  用户提问 → LangChain ↓ 1. Embedding 模型将问题转为向量 ↓ 2. Easysearch 向量检索（kNN）找到相关文档 ↓ 3. 将相关文档 + 问题发送给 LLM ↓ 4. LLM 生成基于上下文的回答 ↓ 用户得到答案 安装 #  pip install langchain langchain-community elasticsearch 连接 Easysearch #  from elasticsearch import Elasticsearch es = Elasticsearch( hosts=[&#34;https://localhost:9200&#34;], basic_auth=(&#34;admin&#34;, &#34;your-password&#34;), verify_certs=False # 自签名证书时使用 ) # 验证连接 print(es.info()) 作为 Vector Store 使用 #  1."
---


# LangChain 集成

[LangChain](https://python.langchain.com/) 是最流行的 LLM 应用开发框架。通过将 Easysearch 作为 Vector Store，可以构建 RAG（Retrieval-Augmented Generation）应用，让大模型基于企业知识库进行问答。

## 架构概览

```
用户提问 → LangChain
              ↓
  1. Embedding 模型将问题转为向量
              ↓
  2. Easysearch 向量检索（kNN）找到相关文档
              ↓
  3. 将相关文档 + 问题发送给 LLM
              ↓
  4. LLM 生成基于上下文的回答
              ↓
用户得到答案
```

## 安装

```bash
pip install langchain langchain-community elasticsearch
```

## 连接 Easysearch

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(
    hosts=["https://localhost:9200"],
    basic_auth=("admin", "your-password"),
    verify_certs=False  # 自签名证书时使用
)

# 验证连接
print(es.info())
```

## 作为 Vector Store 使用

### 1. 准备 Embedding 模型

```python
from langchain_community.embeddings import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="BAAI/bge-base-zh-v1.5"  # 中文 embedding 模型
)
```

### 2. 创建 Vector Store

```python
from langchain_community.vectorstores import ElasticsearchStore

vector_store = ElasticsearchStore(
    es_connection=es,
    index_name="langchain-docs",
    embedding=embeddings,
)
```

### 3. 写入文档

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import TextLoader

# 加载文档
loader = TextLoader("knowledge_base.txt", encoding="utf-8")
documents = loader.load()

# 分块
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
docs = text_splitter.split_documents(documents)

# 写入 Easysearch
vector_store.add_documents(docs)
```

### 4. 相似度搜索

```python
results = vector_store.similarity_search(
    query="Easysearch 如何配置安全？",
    k=5
)
for doc in results:
    print(doc.page_content[:100])
```

## 构建 RAG 问答链

```python
from langchain.chains import RetrievalQA
from langchain_community.llms import Ollama  # 或使用其他 LLM

# 初始化 LLM
llm = Ollama(model="qwen2.5")  # 本地部署的模型

# 构建 RAG 链
retriever = vector_store.as_retriever(search_kwargs={"k": 5})
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    return_source_documents=True
)

# 提问
response = qa_chain.invoke({"query": "Easysearch 的安全功能有哪些？"})
print(response["result"])
```

## 混合搜索（关键词 + 向量）

```python
vector_store = ElasticsearchStore(
    es_connection=es,
    index_name="langchain-docs",
    embedding=embeddings,
    strategy=ElasticsearchStore.ApproxRetrievalStrategy(
        hybrid=True  # 启用混合搜索
    )
)
```

## 注意事项

| 注意项 | 说明 |
|--------|------|
| Embedding 维度 | 需与 `knn_dense_float_vector` 字段的 `dims` 参数匹配 |
| HTTPS | Easysearch 默认启用 HTTPS，注意证书配置 |
| API 兼容 | 开启 `elasticsearch.api_compatibility: true` |
| kNN 插件 | 确保已安装 knn 插件 |

## 延伸阅读

- [RAG 与 LLM 集成]({{< relref "/docs/integrations/ai/rag-and-llm.md" >}})
- [向量搜索]({{< relref "/docs/features/vector-search/_index.md" >}})
- [AI 集成总览]({{< relref "/docs/integrations/ai/" >}})

