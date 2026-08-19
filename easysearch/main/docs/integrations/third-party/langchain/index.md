---
title: "LangChain 集成"
date: 0001-01-01
description: "LangChain 与 Easysearch 的 RAG 集成。"
summary: "LangChain 集成 #   LangChain 是最流行的 LLM 应用开发框架。通过将 Easysearch 作为 Vector Store，可以构建 RAG（Retrieval-Augmented Generation）应用，让大模型基于企业知识库进行问答。
架构概览 #  用户提问 → LangChain ↓ 1. Embedding 模型将问题转为向量 ↓ 2. Easysearch 向量检索（kNN）找到相关文档 ↓ 3. 将相关文档 + 问题发送给 LLM ↓ 4. LLM 生成基于上下文的回答 ↓ 用户得到答案 安装 #  pip install langchain langchain-community elasticsearch 连接 Easysearch #  from elasticsearch import Elasticsearch es = Elasticsearch( hosts=[&#34;https://localhost:9200&#34;], basic_auth=(&#34;admin&#34;, &#34;your-password&#34;), verify_certs=False # 自签名证书时使用 ) # 验证连接 print(es.info()) 作为 Vector Store 使用 #  Easysearch 2."
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

Easysearch 2.4.0 的原生向量索引使用 `dense_vector` 和显式 `index_options.type: hnsw`，不需要安装 k-NN 插件。
`ElasticsearchStore` 可能按客户端默认规则自动建 mapping，不要依赖它生成 Easysearch 2.4.0 要求的显式 HNSW mapping。
先按 [原生 HNSW 搜索]({{< relref "/docs/features/vector-search/native-hnsw.md" >}}) 创建索引，再让 LangChain 写入已有索引。

节点需要开启 Elasticsearch 8.19 兼容模式：

```yaml
elasticsearch.api_compatibility: true
elasticsearch.api_compatibility_version: "8.19.17"
```

### 1. 准备 Embedding 模型

```python
from langchain_community.embeddings import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="BAAI/bge-base-zh-v1.5"  # 中文 embedding 模型
)
```

### 2. 预先创建原生 HNSW 索引

`dims` 必须与 Embedding 模型输出一致。下面以 768 维为例：

```json
PUT /langchain-docs
{
  "mappings": {
    "properties": {
      "text": {
        "type": "text"
      },
      "vector": {
        "type": "dense_vector",
        "dims": 768,
        "index": true,
        "similarity": "cosine",
        "index_options": {
          "type": "hnsw",
          "m": 16,
          "ef_construction": 100
        }
      }
    }
  }
}
```

### 3. 连接已有索引

```python
from langchain_community.vectorstores import ElasticsearchStore

vector_store = ElasticsearchStore(
    es_connection=es,
    index_name="langchain-docs",
    embedding=embeddings,
    query_field="text",
    vector_query_field="vector",
)
```

### 4. 写入文档

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

### 5. 相似度搜索

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

## 复合检索（关键词 + 向量）

Easysearch 2.4.0 **不支持** `ElasticsearchStore.ApproxRetrievalStrategy(hybrid=True)`。该策略会发送并列的
顶层 `knn` 和 `query`，并可能带上 Elasticsearch `rank.rrf`；Easysearch 2.4.0 会拒绝这种请求。

需要关键词和向量复合检索时，使用 Elasticsearch Python 客户端发送 query-level `knn` + `bool` 查询。
查询向量由与写入时相同的 Embedding 模型生成：

```python
query_text = "Easysearch 如何配置安全？"
query_vector = embeddings.embed_query(query_text)

response = es.search(
    index="langchain-docs",
    size=5,
    query={
        "bool": {
            "must": {
                "knn": {
                    "field": "vector",
                    "query_vector": query_vector,
                    "k": 5,
                    "num_candidates": 50,
                }
            },
            "should": {
                "match": {
                    "text": {
                        "query": query_text,
                        "boost": 0.3,
                    }
                }
            },
        }
    },
)
```

这是复合查询，不是 Easysearch 的 [混合搜索]({{< relref "/docs/integrations/ai/hybrid-search.md" >}})。
各路分数不可直接比较、需要 RRF 融合时，使用 Easysearch `hybrid` 查询和 `hybrid_ranker_processor` 搜索管道。

## 注意事项

| 注意项 | 说明 |
|--------|------|
| 字段类型 | 预先创建 `dense_vector`，并显式设置 `index: true` 和 `index_options.type: hnsw` |
| Embedding 维度 | 必须与 `dense_vector` 的 `dims` 以及模型输出一致 |
| HTTPS | Easysearch 默认启用 HTTPS，注意证书配置 |
| API 兼容 | 开启 `elasticsearch.api_compatibility: true`，HNSW 工作负载使用 `"8.19.17"` |
| LangChain Hybrid | 不要使用 `ApproxRetrievalStrategy(hybrid=True)`；Easysearch 2.4.0 不支持它生成的顶层组合请求 |
| 旧 k-NN 插件 | 新索引不需要安装 k-NN 插件；不要把字段建成 `knn_dense_float_vector` |

## 延伸阅读

- [RAG 与 LLM 集成]({{< relref "/docs/integrations/ai/rag-and-llm.md" >}})
- [向量搜索]({{< relref "/docs/features/vector-search/_index.md" >}})
- [AI 集成总览]({{< relref "/docs/integrations/ai/" >}})

