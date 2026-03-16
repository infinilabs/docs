---
title: "语义查询增强处理器"
date: 0001-01-01
summary: "语义查询增强处理器（Semantic Query Enricher Processor） #   需要 AI 插件
 semantic_query_enricher 查询重写处理器用于拦截搜索请求，自动为其中的语义查询（semantic / hybrid 等基于 Embedding 模型的查询）注入模型连接信息（URL、API Key、vendor、model ID），使用户在编写查询时无需手动指定这些参数。
工作原理 #  当搜索请求到达管道时，处理器会遍历查询树，找到所有继承自 ModelBaseQueryBuilder 的查询节点（如 semantic 查询、hybrid 查询中的语义子查询），并为它们绑定 Embedding 请求参数：
 从处理器配置中读取 url、vendor、api_key 根据 default_model_id 或 vector_field_model_id 为每个语义查询字段分配模型 ID 用户只需在查询中写 semantic 查询和查询文本，无需关心模型连接细节  请求体字段 #     字段 类型 是否必填 说明     url String 是 Embedding 模型服务的 URL   vendor String 是 模型提供商标识（如 openai、dashscope、ollama 等）   api_key String 否 模型服务的 API Key。存储时会自动加密   default_model_id String 条件必填 默认模型 ID，应用于所有未指定模型的语义查询字段。与 vector_field_model_id 至少提供一个   vector_field_model_id Object 条件必填 按向量字段指定模型 ID 的映射。格式：{&quot;field_name&quot;: &quot;model_id&quot;}。与 default_model_id 至少提供一个   tag String 否 处理器标识标签   description String 否 处理器描述   ignore_failure Boolean 否 处理器失败时是否继续执行。默认 false    示例 #  使用全局默认模型 #  PUT /_search/pipeline/my_semantic_pipeline { &#34;rewrite_processors&#34;: [ { &#34;semantic_query_enricher&#34;: { &#34;tag&#34;: &#34;enricher&#34;, &#34;description&#34;: &#34;为语义查询注入 Embedding 模型信息&#34;, &#34;url&#34;: &#34;https://dashscope."
---


# 语义查询增强处理器（Semantic Query Enricher Processor）

> **需要 AI 插件**

`semantic_query_enricher` 查询重写处理器用于拦截搜索请求，自动为其中的语义查询（`semantic` / `hybrid` 等基于 Embedding 模型的查询）注入模型连接信息（URL、API Key、vendor、model ID），使用户在编写查询时无需手动指定这些参数。

## 工作原理

当搜索请求到达管道时，处理器会遍历查询树，找到所有继承自 `ModelBaseQueryBuilder` 的查询节点（如 `semantic` 查询、`hybrid` 查询中的语义子查询），并为它们绑定 Embedding 请求参数：

1. 从处理器配置中读取 `url`、`vendor`、`api_key`
2. 根据 `default_model_id` 或 `vector_field_model_id` 为每个语义查询字段分配模型 ID
3. 用户只需在查询中写 `semantic` 查询和查询文本，无需关心模型连接细节

## 请求体字段

| 字段 | 类型 | 是否必填 | 说明 |
|------|------|---------|------|
| `url` | String | 是 | Embedding 模型服务的 URL |
| `vendor` | String | 是 | 模型提供商标识（如 `openai`、`dashscope`、`ollama` 等） |
| `api_key` | String | 否 | 模型服务的 API Key。存储时会自动加密 |
| `default_model_id` | String | 条件必填 | 默认模型 ID，应用于所有未指定模型的语义查询字段。与 `vector_field_model_id` 至少提供一个 |
| `vector_field_model_id` | Object | 条件必填 | 按向量字段指定模型 ID 的映射。格式：`{"field_name": "model_id"}`。与 `default_model_id` 至少提供一个 |
| `tag` | String | 否 | 处理器标识标签 |
| `description` | String | 否 | 处理器描述 |
| `ignore_failure` | Boolean | 否 | 处理器失败时是否继续执行。默认 `false` |

## 示例

### 使用全局默认模型

```json
PUT /_search/pipeline/my_semantic_pipeline
{
  "rewrite_processors": [
    {
      "semantic_query_enricher": {
        "tag": "enricher",
        "description": "为语义查询注入 Embedding 模型信息",
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "vendor": "openai",
        "api_key": "sk-xxxxxxxx",
        "default_model_id": "text-embedding-v3"
      }
    }
  ]
}
```

配置管道后，用户查询只需写：

```json
GET /my_index/_search?search_pipeline=my_semantic_pipeline
{
  "query": {
    "semantic": {
      "embedding": {
        "query_text": "如何配置集群安全"
      }
    }
  }
}
```

处理器会自动为 `semantic` 查询注入 Embedding 服务连接信息。

### 按字段指定不同模型

当索引中不同向量字段使用不同 Embedding 模型时：

```json
PUT /_search/pipeline/multi_model_pipeline
{
  "rewrite_processors": [
    {
      "semantic_query_enricher": {
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "vendor": "openai",
        "api_key": "sk-xxxxxxxx",
        "vector_field_model_id": {
          "title_embedding": "text-embedding-v3",
          "content_embedding": "text-embedding-v2"
        }
      }
    }
  ]
}
```

## 相关文档

- [AI 文本嵌入搜索]({{< relref "/docs/integrations/ai/ai-api/search-text-embedding.md" >}})：完整的语义搜索工作流
- [混合搜索]({{< relref "/docs/integrations/ai/ai-api/hybrid-search.md" >}})：`semantic_query_enricher` 与 `hybrid_ranker_processor` 配合使用

