---
title: "混合搜索"
date: 0001-01-01
summary: "混合搜索 #  混合搜索结合了关键词搜索和语义搜索，以提升搜索相关性。要实现混合搜索，您需要建立一个在搜索时运行的搜索管道。该管道会在中间阶段拦截搜索结果，并通过处理流程对文档分数进行归一化和组合处理。
要使用混合搜索，您需要配置搜索管道，添加混合排序处理器。
相关指南（先读这些） #    向量搜索  搜索管道  混合排序处理器 #  混合排序处理器 hybrid_ranker_processor 是一种重排序处理器，运行在搜索执行的查询阶段和获取阶段之间。它会拦截查询阶段的结果，然后使用 倒数排序融合算法（RRF，Reciprocal Rank Fusion） 来合并不同查询子句，最终生成排序后的搜索结果列表。
适用场景：
 需要融合不同搜索技术（如关键词和语义搜索）的结果 子查询的原始分数不可直接比较（如 BM25 和 KNN）  算法原理 #  RRF 是一种多查询融合方法，其核心计算逻辑为：
  对每个文档在不同子结果集给出的排名取倒数（如排名第 k 则得分为 1/(k+60)） 将各子结果集的倒数得分相加，生成统一排序分数 按最终分数降序输出结果集   RRF 的通用计算公式如下（其中 k 为平滑常数，默认 60，query_j_rank 表示混合查询中某文档在第 j 种查询方法返回结果中的排名）：
rankScore(document_i) = sum(1/(k + query_1_rank), 1/(k + query_2_rank), ..., 1/(k + query_j_rank)) 请求体字段 #  下表列出了所有可用的请求字段。"
---


# 混合搜索

混合搜索结合了关键词搜索和语义搜索，以提升搜索相关性。要实现混合搜索，您需要建立一个在搜索时运行的搜索管道。该管道会在中间阶段拦截搜索结果，并通过处理流程对文档分数进行归一化和组合处理。

要使用混合搜索，您需要配置搜索管道，添加混合排序处理器。

## 相关指南（先读这些）

- [向量搜索]({{< relref "/docs/features/vector-search/_index.md" >}})
- [搜索管道]({{< relref "/docs/features/query-dsl/search-pipelines/_index.md" >}})

## 混合排序处理器
混合排序处理器 `hybrid_ranker_processor` 是一种重排序处理器，运行在搜索执行的查询阶段和获取阶段之间。它会拦截查询阶段的结果，然后使用 **倒数排序融合算法（RRF，Reciprocal Rank Fusion）** 来合并不同查询子句，最终生成排序后的搜索结果列表。

适用场景：
- 需要融合不同搜索技术（如关键词和语义搜索）的结果
- 子查询的原始分数不可直接比较（如 BM25 和 KNN）

### 算法原理

RRF 是一种多查询融合方法，其核心计算逻辑为：
> 1. 对每个文档在不同子结果集给出的排名取倒数（如排名第 k 则得分为 1/(k+60)）
> 2. 将各子结果集的倒数得分相加，生成统一排序分数
> 3. 按最终分数降序输出结果集

RRF 的通用计算公式如下（其中 k 为平滑常数，默认 60，query_j_rank 表示混合查询中某文档在第 j 种查询方法返回结果中的排名）：
```math
rankScore(document_i) = sum(1/(k + query_1_rank), 1/(k + query_2_rank), ..., 1/(k + query_j_rank))
```

### **请求体字段**
下表列出了所有可用的请求字段。

| 字段 | 数据类型 | 说明 |
|------|----------|------|
| `combination.technique` | String | **必填**。指定分数组合方式，目前仅支持 `rrf`（倒数排序融合）。 |
| `combination.rank_constant` | Integer | **可选**。计算倒数得分前，加到文档排名的常量值（必须 ≥ 1）。默认值：60。<br> - **较大值**（如 100）会使分数更均匀，降低高排名结果的影响。<br> - **较小值**（如 10）会增大排名间的分数差异，使高排名结果更具优势。 |

（注：RRF 算法适用于混合搜索场景，能够平衡关键词搜索和语义搜索的排序结果，提升整体相关性。）


### 创建混合排序处理器
以下请求创建一个搜索管道，其中包含混合排序处理器：
```auto
PUT /_search/pipeline/rrf-pipeline
{
  "rerank_processors": [
    {
      "hybrid_ranker_processor": {
        "combination": {
          "technique": "rrf",
          "rank_constant": 60
        }
      }
    }
  ]
}
```
rrf-pipeline 仅用于演示，实际使用时建议配置混合搜索管道

## 配置混合搜索管道
为充分发挥混合搜索的优势，建议配置语义查询增强处理器 `semantic_query_enricher`，通过结合关键词搜索的精确匹配能力和语义搜索的上下文理解能力，提升整体搜索效果。

下面这个请求是在 Easysearch 中创建一个名为 search_model_aliyun 的搜索管道(search pipeline)。搜索管道允许你在搜索请求的不同阶段插入自定义逻辑。
```auto
PUT /_search/pipeline/search_model_aliyun
{
  "rewrite_processors": [
    {
      "semantic_query_enricher" : {
        "tag": "tag1",
        "description": "aliyun search embedding model",
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1/embeddings",
        "vendor": "openai",
        "api_key": "<api_key>",
        "default_model_id": "text-embedding-v4",
        "vector_field_model_id": {
           "text_vector": "text-embedding-v4"
        }
      }
    }
  ],
     "rerank_processors": [
    {
      "hybrid_ranker_processor": {
        "combination": {
          "technique": "rrf",
          "rank_constant": 60
        }
      }
    }
  ],
    "enrich_processors": [
    {
        "hybrid_score_explanation": {}
    }
  ]
}
```
### 搜索管道结构：

#### 查询重写处理器 (rewrite_processors)
```auto
{
   "semantic_query_enricher": {}
}
```
这部分配置了一个语义查询增强器，将查询文本转换为向量表示，用于后续的向量搜索。
#### 重排序处理器 (rerank_processors)
```auto
{
   "hybrid_ranker_processor": {}
}
```
这部分配置了一个混合排序处理器。
#### 结果增强处理器 (enrich_processors)，可选
```auto
{
   "hybrid_score_explanation": {}
}
```
这部分配置了一个混合分数解释器，它会：
- 在搜索结果中添加解释信息
- 帮助理解不同部分(如文本匹配和语义相似度)对最终得分的贡献

## 使用混合查询 API 进行搜索
将 search_model_aliyun 设置为默认搜索管道
 ```auto
 PUT /my-index/_settings
 {
   "index.search.default_pipeline" : "search_model_aliyun"
 }
 ```

执行混合搜索
以下示例请求组合了两个查询子句：[语义查询]({{< relref "/docs/integrations/ai/ai-api/search-text-embedding.md" >}})和匹配查询。它使用上一步设置的默认搜索管道进行查询
 ```auto
 GET /my-index/_search
 {
   "_source": {
     "exclude": [
       "text_vector"
     ]
   },
   "query": {
     "hybrid": {
       "queries": [
         {
           "match": {
             "input_text": "夏季旅游首选"
           }
         },
         {
           "semantic": {
             "text_vector": {
               "query_text": "夏季旅游首选",
               "candidates": 10,
               "query_strategy": "LSH_COSINE"
             }
           }
         }
       ]
     }
   }
 }
 ```
