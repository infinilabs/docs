---
title: "NLP 自然语言处理"
date: 0001-01-01
summary: "NLP 自然语言处理 #  搜索引擎的核心挑战是理解人类语言。本文介绍 NLP（Natural Language Processing）在 Easysearch 中的应用，从基础的分词到高级的向量语义搜索。
NLP 在搜索中的角色 #  用户输入: &#34;我想买一台便宜的笔记本电脑&#34; ↓ NLP 处理层（分词、去停用词、同义词、向量化） ↓ Easysearch 查询执行 ↓ 返回相关结果（包括 &#34;laptop&#34;、&#34;notebook&#34;、&#34;手提电脑&#34; 等） Easysearch 中 NLP 的应用可以分为三个层次：
   层次 技术 实现方式     词级处理 分词、词干提取、停用词 内置分析器   语言规则 同义词、拼音、模糊匹配 分析器插件   语义理解 向量检索、文本嵌入 AI 集成 / kNN    第一层：分词与文本分析 #  分词是 NLP 的基础——将连续文本切分为独立的词项（Token）。
中文分词 #  中文没有空格分隔，需要专用分词器：
PUT /my-index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;ik_smart_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;ik_smart&#34; } } } } } 测试分词效果："
---


# NLP 自然语言处理

搜索引擎的核心挑战是理解人类语言。本文介绍 NLP（Natural Language Processing）在 Easysearch 中的应用，从基础的分词到高级的向量语义搜索。

## NLP 在搜索中的角色

```
用户输入: "我想买一台便宜的笔记本电脑"
              ↓
  NLP 处理层（分词、去停用词、同义词、向量化）
              ↓
  Easysearch 查询执行
              ↓
  返回相关结果（包括 "laptop"、"notebook"、"手提电脑" 等）
```

Easysearch 中 NLP 的应用可以分为三个层次：

| 层次 | 技术 | 实现方式 |
|------|------|----------|
| **词级处理** | 分词、词干提取、停用词 | 内置分析器 |
| **语言规则** | 同义词、拼音、模糊匹配 | 分析器插件 |
| **语义理解** | 向量检索、文本嵌入 | AI 集成 / kNN |

## 第一层：分词与文本分析

分词是 NLP 的基础——将连续文本切分为独立的词项（Token）。

### 中文分词

中文没有空格分隔，需要专用分词器：

```
PUT /my-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "ik_smart_analyzer": {
          "type": "custom",
          "tokenizer": "ik_smart"
        }
      }
    }
  }
}
```

测试分词效果：

```
POST /_analyze
{
  "analyzer": "ik_smart",
  "text": "INFINI Easysearch 是分布式搜索引擎"
}

# 输出词项: ["infini", "easysearch", "是", "分布式", "搜索引擎"]
```

Easysearch 内置的 IK 分析器支持两种粒度：

| 分词器 | 粒度 | 示例 |
|--------|------|------|
| `ik_smart` | 粗粒度（最少切分） | "中华人民共和国" → ["中华人民共和国"] |
| `ik_max_word` | 细粒度（最多切分） | "中华人民共和国" → ["中华人民共和国", "中华人民", "中华", "人民共和国", "人民", "共和国"] |

> 详见 [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})。

### 词干提取（Stemming）

将单词还原到词干形式，提高召回率：

```
running → run
searches → search
better → better (不规则变化需要词典)
```

> 详见 [词干化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})。

### 停用词

去除对搜索无意义的高频词（"的"、"是"、"the"、"a"）：

```json
{
  "analysis": {
    "filter": {
      "my_stop": {
        "type": "stop",
        "stopwords": ["的", "了", "在", "是", "我"]
      }
    }
  }
}
```

> 详见 [停用词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stopwords.md" >}})。

## 第二层：语言增强

### 同义词

让 "手机"、"手提电话"、"mobile phone" 互相匹配：

```json
{
  "analysis": {
    "filter": {
      "my_synonyms": {
        "type": "synonym",
        "synonyms": [
          "手机,手提电话,mobile phone",
          "笔记本,laptop,notebook"
        ]
      }
    }
  }
}
```

> 详见 [同义词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-synonyms.md" >}})。

### 拼音搜索

通过拼音插件，支持拼音首字母和全拼搜索：

```
"刘德华" → 可通过 "ldh" 或 "liudehua" 搜索到
```

### 模糊匹配

容忍拼写错误：

```json
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "esysearch",
        "fuzziness": 1
      }
    }
  }
}
```

> 详见 [模糊搜索]({{< relref "/docs/features/mapping-and-analysis/text-analysis-fuzzy.md" >}})。

## 第三层：语义搜索

传统关键词搜索依赖词项精确匹配，无法理解语义。语义搜索通过向量嵌入（Embedding）解决这个问题。

### 基本流程

```
文本 → Embedding 模型 → 向量 [0.12, -0.34, 0.56, ...]
                              ↓
               存入 Easysearch knn_dense_float_vector 字段
                              ↓
             使用 knn_nearest_neighbors 查询最近邻
```

### 向量字段定义

```json
PUT /semantic-index
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "title_vector": {
        "type": "knn_dense_float_vector",
        "knn": {
          "dims": 768,
          "model": "lsh",
          "similarity": "cosine",
          "L": 99,
          "k": 1
        }
      }
    }
  }
}
```

### kNN 搜索

```json
GET /semantic-index/_search
{
  "query": {
    "knn_nearest_neighbors": {
      "field": "title_vector",
      "vec": {
        "values": [0.12, -0.34, ...]
      },
      "model": "lsh",
      "similarity": "cosine",
      "candidates": 50
    }
  }
}
```

### 混合搜索

将关键词搜索与向量搜索结合，兼顾精确性和语义理解：

```json
GET /my-index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "knn_nearest_neighbors": {
            "field": "title_vector",
            "vec": { "values": [...] },
            "model": "lsh",
            "similarity": "cosine",
            "candidates": 50
          }
        }
      ],
      "should": [
        { "match": { "title": "搜索引擎" } }
      ]
    }
  }
}
```

> 詳見 [向量搜索]({{< relref "/docs/features/vector-search/_index.md" >}}) 和 [AI 集成]({{< relref "/docs/integrations/ai/" >}})。

## 选型建议

| 场景 | 推荐方案 | 复杂度 |
|------|----------|:------:|
| 中文关键词搜索 | IK 分词 + 同义词 | 低 |
| 多语言搜索 | 对应语言的分析器 | 低 |
| 拼写纠错 | fuzzy + suggest | 中 |
| 语义搜索 / 问答 | Embedding + kNN | 高 |
| 最佳效果 | 混合搜索（关键词 + 向量） | 高 |

## 延伸阅读

- [文本分析]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [同义词]({{< relref "/docs/features/mapping-and-analysis/text-analysis-synonyms.md" >}})
- [向量搜索]({{< relref "/docs/features/vector-search/_index.md" >}})
- [AI 集成]({{< relref "/docs/integrations/ai/" >}})
- [RAG 与 LLM]({{< relref "/docs/integrations/ai/rag-and-llm.md" >}})

### 最佳实践

- [AI 搜索与向量检索架构]({{< relref "/docs/best-practices/ai-search-architecture.md" >}})：多路召回、Hybrid 搜索、LLM 集成
- [向量字段建模]({{< relref "/docs/best-practices/data-modeling/vector-fields.md" >}})：向量维度选择、写入策略、成本控制
- [查询调优]({{< relref "/docs/best-practices/query-tuning.md" >}})：NLP 查询的性能优化

