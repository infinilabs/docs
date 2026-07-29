---
title: "向量字段建模指南"
date: 0001-01-01
description: "语义搜索中的向量字段设计、存储优化与性能权衡。覆盖字段设计、维度选择、多向量模式、写入策略等关键决策。"
summary: "向量字段建模指南 #  概述 #  向量字段是实现语义搜索（向量/kNN 搜索）的基础设施。本指南覆盖 Easysearch 中向量字段的设计原则、存储优化、性能权衡，帮助你做出合理的架构决策。
核心出发点： 向量字段的设计直接影响三个维度——存储成本、索引性能、查询效率。在业务约束下找到最优平衡是关键。
 1. 文档模型设计 #  1.1 基本模式：混合字段设计 #  在支持语义搜索的系统中，典型文档包含三类字段：
{ &#34;_id&#34;: &#34;doc-001&#34;, &#34;metadata&#34;: { &#34;title&#34;: &#34;Easysearch 向量搜索最佳实践&#34;, &#34;created_at&#34;: &#34;2026-02-13&#34;, &#34;category&#34;: &#34;AI搜索&#34; }, &#34;content&#34;: &#34;完整的正文内容……&#34;, &#34;embedding&#34;: [0.124, -0.031, 0.092, ...], &#34;snippet&#34;: &#34;用于快速展示的摘要&#34; } 字段分工：
   字段类型 典型字段名 数据类型 用途 存储开销     文本字段 title, content text BM25 全文搜索、分面过滤 低   元数据字段 created_at, category keyword, date 精确过滤、排序、聚合 低   向量字段 embedding, vector dense_vector kNN 语义相似度查询 高   展示字段 snippet, summary text（不分词） 快速返回、避免重查询 中    设计要点："
---


# 向量字段建模指南

## 概述

向量字段是实现语义搜索（向量/kNN 搜索）的基础设施。本指南覆盖 Easysearch 中向量字段的设计原则、存储优化、性能权衡，帮助你做出合理的架构决策。

**核心出发点：** 向量字段的设计直接影响三个维度——存储成本、索引性能、查询效率。在业务约束下找到最优平衡是关键。

---

## 1. 文档模型设计

### 1.1 基本模式：混合字段设计

在支持**语义搜索**的系统中，典型文档包含三类字段：

```json
{
  "_id": "doc-001",
  "metadata": {
    "title": "Easysearch 向量搜索最佳实践",
    "created_at": "2026-02-13",
    "category": "AI搜索"
  },
  "content": "完整的正文内容……",
  "embedding": [0.124, -0.031, 0.092, ...],
  "snippet": "用于快速展示的摘要"
}
```

**字段分工：**

| 字段类型 | 典型字段名 | 数据类型 | 用途 | 存储开销 |
|---------|----------|--------|------|---------|
| 文本字段 | `title`, `content` | `text` | BM25 全文搜索、分面过滤 | 低 |
| 元数据字段 | `created_at`, `category` | `keyword`, `date` | 精确过滤、排序、聚合 | 低 |
| 向量字段 | `embedding`, `vector` | `dense_vector` | kNN 语义相似度查询 | **高** |
| 展示字段 | `snippet`, `summary` | `text`（不分词） | 快速返回、避免重查询 | 中 |

**设计要点：**
- 文本字段和向量字段**并存**，支持混合查询（BM25 + kNN）
- 向量字段**不需要分词、不需要倒排索引**，但需要特殊的向量索引（如 HNSW）
- 避免冗余存储：已有 `embedding` 时，一般不需要额外的 `content_vector` 除非有多种语义模式

### 1.2 映射定义示例

```json
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      },
      "content": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "category": {
        "type": "keyword"
      },
      "created_at": {
        "type": "date"
      },
      "embedding": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,
        "similarity": "cosine"
      },
      "snippet": {
        "type": "text",
        "index": false,
        "enabled": true
      }
    }
  }
}
```

---

## 2. 维度选择与成本控制

### 2.1 维度对成本的影响

向量维度（`dims`）是影响系统资源消耗的最重要因素：

```
单条文档开销 ≈ dims × 4 字节（float32）

示例：
- 维度 128  →  512 字节/文档
- 维度 384  →  1.5 KB/文档
- 维度 768  →  3 KB/文档
- 维度 1536 →  6 KB/文档（如 OpenAI 的 embedding）
```

**100 万文档的存储对比：**

| 维度 | 向量存储大小 | 索引开销 | 总体影响 |
|-----|-----------|--------|--------|
| 128 | ~512 MB | 低 | ✅ 极优 |
| 384 | ~1.5 GB | 中 | ✅ 推荐 |
| 768 | ~3 GB | 高 | ⚠️ 中等 |
| 1536 | ~6 GB | 很高 | ⚠️ 需评估 |

### 2.2 维度选择策略

**优先级原则（从高到低）：**

1. **业务精度要求** → 必须满足搜索效果
2. **成本-效果平衡** → 在保证效果的前提下最小化维度
3. **基础设施约束** → 内存、存储、网络带宽限制

**实践建议：**

```
场景 1：通用语义搜索（推荐）
└─ 使用 384 维模型（如 sentence-transformers 系列）
   └─ 成本合理 + 效果充分
   
场景 2：精细领域应用（金融、医疗）
└─ 使用 768 维模型
   └─ 更强的表达能力，但需评估存储成本
   
场景 3：大规模系统（超 5000 万文档）
└─ 使用 128～256 维压缩模型
   └─ 或使用量化（binary/int8）模式（如果引擎支持）

场景 4：超大维度（>1000）
└─ 评估是否真的必要
└─ 考虑降维：PCA、LSH 等预处理
└─ 或采用多阶段检索：粗排用低维，精排用高维
```

### 2.3 动态维度策略

对于**演进中的系统**，可采用多字段策略：

```json
{
  "properties": {
    "embedding_v1": {
      "type": "dense_vector",
      "dims": 384,
      "index": true,    // 线上用
      "similarity": "cosine"
    },
    "embedding_v2": {
      "type": "dense_vector",
      "dims": 768,
      "index": false,   // 过渡阶段，暂不用于查询
      "similarity": "cosine"
    }
  }
}
```

**好处：** 逐步升级模型，减少迁移成本。  
**成本：** 额外的存储开销（短期）。

---

## 3. 多向量模式

### 3.1 何时需要多向量

**场景判断：**

| 场景 | 是否需要多向量 | 说明 |
|-----|------------|------|
| 单一搜索场景（通用搜索） | ❌ 否 | 一个 embedding 足够 |
| 多语言文档库 | ✅ 是 | 不同语言用不同向量模型 |
| 标题和正文语义不同 | ⚠️ 可选 | 取决于是否分别查询 |
| 不同时间段数据用不同模型 | ⚠️ 可选 | 如无必要，应该做离线重算 |
| A/B 测试新模型 | ✅ 是（临时） | 双向量并行验证，验证后清理 |

### 3.2 多向量设计示例

```json
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" },
      "title_embedding": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,
        "similarity": "cosine"
      },
      "content_embedding": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,
        "similarity": "cosine"
      }
    }
  }
}
```

**查询时：**
```python
# 基于标题的语义查询
response = es.search(
    index="docs",
    body={
        "query": {
            "knn": {
                "title_embedding": {
                    "vector": [0.12, -0.03, ...],
                    "k": 10
                }
            }
        }
    }
)

# 或使用 bool 组合（标题权重更高）
# 详见搜索章节的 Hybrid 检索指南
```

### 3.3 多向量的成本警告

⚠️ **每增加一个向量字段，成本翻倍：**

```
2 个向量字段 = 2× 存储 + 2× 索引构建时间 + 2× 内存占用
```

**决策框架：**

```
是否真的必须分别查询这两个向量？
  ↓
  是 → 需要多向量
  ↓
  否 → 考虑：
      a) 用单向量拼接/融合（如 concat([title_embedding, content_embedding]))
      b) 用 boosting 给标题更高权重（在查询侧解决）
      c) 在应用层做分步检索（先查标题，再查内容）
```

---

## 4. 写入与数据一致性

### 4.1 同步 vs 异步策略

**策略对比：**

| 方案 | 延迟 | 一致性 | 模型服务负载 | 适用场景 |
|-----|-----|------|-----------|--------|
| **同步生成** | 高（秒级） | ✅ 强一致 | 高 | 对搜索精度敏感的应用 |
| **异步补齐** | 低（毫秒） | ⚠️ 最终一致 | 低 | 实时性要求不高，数据量大 |
| **混合策略** | 中等 | ✅ 可控 | 中等 | **推荐** |

### 4.2 同步写入实现

```python
from elasticsearch import Elasticsearch
import requests

es = Elasticsearch(["localhost:9200"])
embedding_service = "http://localhost:8001"  # 向量服务

def index_document(doc):
    # 1. 调用模型生成向量
    embedding_response = requests.post(
        f"{embedding_service}/embed",
        json={"text": doc["content"]},
        timeout=5
    )
    
    if embedding_response.status_code != 200:
        # 降级策略：生成失败后如何处理？
        # 选项 A：中断写入，返回错误
        # 选项 B：仅写文本，标记为"缺向量"，后续异步补齐
        logger.warning(f"Embedding failed for doc {doc['_id']}, marking for async")
        doc["_embedding_status"] = "pending"
    else:
        doc["embedding"] = embedding_response.json()["vector"]
    
    # 2. 写入 Easysearch
    es.index(
        index="documents",
        id=doc.get("_id"),
        body=doc
    )
```

**注意事项：**
- 设置合理的**超时时间**（建议 5～10 秒）
- 实现**失败重试机制**和**降级策略**
- 监控向量服务的延迟和错误率

### 4.3 异步补齐实现

```python
# 方案：使用 Kafka/消息队列解耦

def index_document_async(doc):
    # 1. 立即写入文档（不含向量）
    es.index(
        index="documents",
        id=doc.get("_id"),
        body={
            **doc,
            "_embedding_status": "pending"
        }
    )
    
    # 2. 发送到异步任务队列
    kafka_producer.send("embedding_tasks", {
        "doc_id": doc["_id"],
        "text": doc["content"]
    })

# 异步 Worker 处理：
def embedding_worker():
    for message in kafka_consumer.consume("embedding_tasks"):
        task = json.loads(message.value)
        
        # 批量处理，提升效率
        embeddings = model.encode([task["text"]])
        
        # 更新文档的向量字段
        es.update(
            index="documents",
            id=task["doc_id"],
            body={
                "doc": {
                    "embedding": embeddings[0],
                    "_embedding_status": "completed"
                }
            }
        )
```

### 4.4 混合策略（推荐）

```
实时性高的业务逻辑 → 同步生成 + 写入
  ↓
后台数据导入 → 异步补齐
  ↓
批量更新 → 定时离线处理
```

**好处：** 在用户体验和系统负载间找到平衡。

---

## 5. 存储与索引优化

### 5.1 禁用不必要的索引

如果**不在线上查询某个向量**，应禁用其索引以节省资源：

```json
{
  "properties": {
    "embedding_online": {
      "type": "dense_vector",
      "dims": 384,
      "index": true,      // ✅ 用于 kNN 查询
      "similarity": "cosine"
    },
    "embedding_offline": {
      "type": "dense_vector",
      "dims": 768,
      "index": false,     // ❌ 仅存储，不索引
      "similarity": "cosine"
    }
  }
}
```

**开销对比（100 万文档）：**
- `index: true` → 存储 + 索引，共约 2.5～3 GB（384 维）
- `index: false` → 仅存储，约 1.5 GB

### 5.2 向量量化（如果支持）

某些搜索引擎支持 **int8/binary 量化**，可进一步节省空间：

```
原始 float32 向量: 4 字节 × 384 = 1.5 KB
量化为 int8:      1 字节 × 384 = 384 字节  (节省 75%)
量化为 binary:    ~48 字节 × 1  = 48 字节  (节省 97%)
```

**权衡：** 精度下降 vs 资源节省。需实际评估对搜索效果的影响。

### 5.3 分片与副本策略

向量索引通常**比文本索引更消耗内存**，需要特别关注集群资源：

```json
{
  "settings": {
    "number_of_shards": 5,      // 根据数据量调整
    "number_of_replicas": 1,    // 高可用性
    "index.store.type": "niofs"  // 使用内存映射，加速 kNN
  }
}
```

**建议：**
- 向量索引的**分片大小**控制在 30～50 GB
- 如果单个向量字段超过 100 GB，考虑**独立分片**或**多索引**

---

## 6. 多向量融合与降维

### 6.1 向量拼接融合

**场景：** 需要同时考虑标题和内容的语义

**方案 1：应用层融合（推荐）**

```python
def fuse_embeddings(title_emb, content_emb, weights=(0.3, 0.7)):
    """
    线性加权融合两个向量
    标题权重 30%，内容权重 70%
    """
    fused = (
        np.array(title_emb) * weights[0] +
        np.array(content_emb) * weights[1]
    )
    # 归一化
    return fused / np.linalg.norm(fused)

# 写入
doc["embedding"] = fuse_embeddings(
    model.encode(doc["title"]),
    model.encode(doc["content"])
)
```

**优点：** 单向量，成本最低；权重可灵活调整。

**方案 2：多向量并行查询（见搜索章节）**

允许用户同时对标题和内容向量查询，系统端融合排序。

### 6.2 PCA 降维

对于维度过高的向量（如 1536），可以用 **PCA 预处理降至 384 维**，保留 95%+ 的信息：

```python
from sklearn.decomposition import PCA

# 离线：对样本向量做降维
pca = PCA(n_components=384)
original_vectors = [...]  # shape: (N, 1536)
reduced_vectors = pca.fit_transform(original_vectors)

# 保存 PCA 模型
joblib.dump(pca, "pca_model.pkl")

# 在线：新来的向量也要用同一个 PCA 模型降维
new_vector = model.encode("some text")  # shape: (1536,)
new_reduced = pca.transform([new_vector])[0]  # shape: (384,)
```

**成本变化：**
- 从 1536 维 → 384 维 ≈ **节省 75% 存储**
- **查询速度提升** 10～15 倍
- **精度损失** 通常在可接受范围（1～3%）

---

## 7. 监控与运维

### 7.1 关键指标

监控以下指标确保系统健康：

```
存储相关：
├─ 单个文档平均大小（向量占比）
├─ 索引总大小 vs 原始数据大小（压缩率）
└─ 磁盘使用趋势

性能相关：
├─ kNN 查询延迟（P50, P99）
├─ 向量索引构建时间
├─ 内存使用率
└─ 向量生成服务的吞吐量和错误率

数据相关：
├─ 缺向量文档数量（`_embedding_status: pending`）
├─ 向量更新延迟
└─ 多向量同步率（如有多个向量字段）
```

### 7.2 告警规则示例

```yaml
告警: 向量生成服务超时率 > 5%
  └─ 行动：扩容服务 / 优化模型推理

告警: 缺向量文档数量 > 文档总数的 1%
  └─ 行动：检查异步补齐任务 / 查看错误日志

告警: kNN 查询 P99 延迟 > 1 秒
  └─ 行动：检查节点负载 / 评估是否需要优化维度
```

---

## 8. 总结与决策树

### 快速决策流程

```
Q1: 需要做向量/语义搜索吗？
├─ 否 → 使用纯文本索引
└─ 是 ↓

Q2: 数据量大小？
├─ < 100 万 → 维度可用 768
├─ 100 万～1000 万 → 推荐 384
└─ > 1000 万 → 谨慎选择，推荐 256～384 + 量化 ↓

Q3: 有多种语义视角吗（标题 vs 内容等）？
├─ 否 → 单向量 + 应用层融合
└─ 是 ↓

Q4: 必须分别查询吗？
├─ 否 → 融合为单向量
└─ 是 → 多向量（但评估成本） ↓

Q5: 向量从何而来？
├─ 实时生成 → 同步写入 + 降级策略
├─ 离线预算 → 异步补齐
└─ 混合 → 混合策略
```

### 核心最佳实践

| 原则 | 说明 |
|-----|------|
| **够用最优** | 在满足效果的前提下最小化维度和字段数 |
| **明确用途** | 每个向量字段都要清楚其查询需求 |
| **异步解耦** | 向量生成不应阻塞主业务流程 |
| **可观测性** | 监控向量生成、存储、查询的全链路 |
| **灰度迁移** | 更换模型时用多字段并行验证，而非全量切换 |

---

## 相关章节

- [向量查询与语义搜索]({{< relref "/docs/features/vector-search/knn_api.md" >}}) - 如何查询向量字段
- [Hybrid 检索]({{< relref "/docs/features/fulltext-search" >}}) - 混合文本和向量搜索
- [字段类型参考]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}}) - dense_vector 字段详细参数
- [Mapping 与文本分析]({{< relref "/docs/fundamentals/mapping-analysis-intro.md" >}}) - 基础概念


