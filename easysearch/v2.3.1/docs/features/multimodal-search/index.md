---
title: "多模态搜索"
date: 0001-01-01
description: "跨越文本边界，支持图片、音频、视频等多种数据形态的统一向量检索。"
summary: "企业中大量数据以图片、视频、音频等非结构化形式存在。Easysearch 多模态搜索通过跨媒介的特征向量匹配，打通不同数据形态之间的检索壁垒——用户可以&quot;以图搜图&quot;、通过文字描述找到匹配的图片，甚至跨越音视频与文本的边界进行关联检索。
 工作原理 #  多模态搜索的核心在于：将不同类型的数据（文本、图片、音频等）通过多模态 Embedding 模型转换为同一向量空间中的向量表示，然后利用 Easysearch 的 向量搜索 能力进行相似度匹配。
图片 ──→ 多模态模型 ──→ 向量 ──┐ ├──→ 向量空间相似度计算 ──→ 排序结果 文本 ──→ 多模态模型 ──→ 向量 ──┘ 关键点：
 不同模态的数据被编码到同一向量空间 向量之间的距离反映跨模态的语义相似度 Easysearch 负责高性能的向量存储与检索   索引设计 #  多模态搜索的索引需要同时存储原始元数据和多模态向量：
PUT /multimodal-assets { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;description&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;media_type&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;file_url&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;tags&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;embedding&#34;: { &#34;type&#34;: &#34;knn_dense_float_vector&#34;, &#34;knn&#34;: { &#34;dims&#34;: 512 } } } } } 写入时，将图片/音频/视频先通过多模态 Embedding 模型（如 CLIP、ImageBind 等）转换为向量："
---


企业中大量数据以图片、视频、音频等非结构化形式存在。Easysearch 多模态搜索通过跨媒介的特征向量匹配，打通不同数据形态之间的检索壁垒——用户可以"以图搜图"、通过文字描述找到匹配的图片，甚至跨越音视频与文本的边界进行关联检索。

---

## 工作原理

多模态搜索的核心在于：**将不同类型的数据（文本、图片、音频等）通过多模态 Embedding 模型转换为同一向量空间中的向量表示**，然后利用 Easysearch 的 [向量搜索]({{< relref "../vector-search" >}}) 能力进行相似度匹配。

```
图片 ──→ 多模态模型 ──→ 向量 ──┐
                                ├──→ 向量空间相似度计算 ──→ 排序结果
文本 ──→ 多模态模型 ──→ 向量 ──┘
```

关键点：

- 不同模态的数据被编码到同一向量空间
- 向量之间的距离反映跨模态的语义相似度
- Easysearch 负责高性能的向量存储与检索

---

## 索引设计

多模态搜索的索引需要同时存储原始元数据和多模态向量：

```json
PUT /multimodal-assets
{
  "mappings": {
    "properties": {
      "description":  { "type": "text" },
      "media_type":   { "type": "keyword" },
      "file_url":     { "type": "keyword" },
      "tags":         { "type": "keyword" },
      "embedding": {
        "type": "knn_dense_float_vector",
        "knn": { "dims": 512 }
      }
    }
  }
}
```

写入时，将图片/音频/视频先通过多模态 Embedding 模型（如 CLIP、ImageBind 等）转换为向量：

```json
POST /multimodal-assets/_doc
{
  "description": "一只金色拉布拉多在草地上奔跑",
  "media_type": "image",
  "file_url": "/assets/dog-running.jpg",
  "tags": ["dog", "outdoor"],
  "embedding": [0.23, -0.11, 0.45, ...]
}
```

---

## 跨模态查询

### 以文搜图

将文字描述转换为向量，在图片向量库中搜索：

```json
POST /multimodal-assets/_search
{
  "size": 5,
  "query": {
    "knn_nearest_neighbors": {
      "field": "embedding",
      "vec": {
        "values": [0.21, -0.08, 0.42, ...]
      },
      "model": "lsh",
      "similarity": "cosine",
      "candidates": 50
    }
  }
}
```

### 以图搜图

将上传的图片通过模型生成向量，再进行搜索：

```json
POST /multimodal-assets/_search
{
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "knn_nearest_neighbors": {
            "field": "embedding",
            "vec": { "values": [0.24, -0.12, 0.44, ...] },
            "model": "lsh",
            "similarity": "cosine",
            "candidates": 100
          }
        }
      ],
      "filter": [
        { "term": { "media_type": "image" } }
      ]
    }
  }
}
```

### 多模态 + 结构化混合查询

在进行跨模态检索时，可同步叠加时间、类型、标签等结构化条件：

```json
POST /multimodal-assets/_search
{
  "size": 10,
  "query": {
    "bool": {
      "must": [
        {
          "knn_nearest_neighbors": {
            "field": "embedding",
            "vec": { "values": [0.21, -0.08, 0.42, ...] },
            "model": "lsh",
            "similarity": "cosine",
            "candidates": 50
          }
        }
      ],
      "filter": [
        { "term": { "media_type": "image" } },
        { "terms": { "tags": ["outdoor", "nature"] } }
      ]
    }
  }
}
```

---

## 常用多模态模型

| 模型 | 维度 | 支持模态 | 说明 |
|------|------|----------|------|
| CLIP (OpenAI) | 512/768 | 文本 + 图片 | 最经典的跨模态模型 |
| Chinese-CLIP | 512/768 | 文本 + 图片 | 中文优化版 CLIP |
| ImageBind (Meta) | 1024 | 文本/图片/音频/视频/IMU | 六模态统一模型 |
| Jina CLIP v2 | 768 | 文本 + 图片 | 多语言、高性能 |

---

## 应用场景

| 场景 | 说明 |
|------|------|
| **智能素材库与版权管理** | 通过上传草图或相似图片快速找回源文件，或进行版权侵权比对 |
| **电商视觉搜索** | 用户拍摄实物照片即可在商城中匹配同款或相似款商品 |
| **公共安全与工业检测** | 在监控视频或工业探伤图像中，通过特征匹配快速锁定目标 |
| **音视频内容风控** | 通过音频指纹或画面特征，在海量视频中快速识别违规内容 |

---

## 相关文档

- [向量搜索]({{< relref "../vector-search" >}})
- [语义搜索]({{< relref "../semantic-search" >}})
- [Embedding 服务集成]({{< relref "../../integrations/ai/embedding-service" >}})
- [向量检索端到端流程]({{< relref "../../integrations/ai/vector-workflow" >}})
