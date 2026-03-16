---
title: "文本向量化处理器"
date: 0001-01-01
summary: "文本向量化处理器 #   需要 AI 插件和 KNN 插件
 text_embedding 处理器在文档写入时自动调用外部 Embedding 模型服务，将文本字段转换为向量并存储到指定的向量字段中，实现&quot;写入即向量化&quot;。该处理器是构建语义搜索和混合搜索的基础组件。
语法 #  { &#34;text_embedding&#34;: { &#34;text_field&#34;: &#34;content&#34;, &#34;vector_field&#34;: &#34;content_vector&#34;, &#34;url&#34;: &#34;https://api.openai.com/v1/embeddings&#34;, &#34;vendor&#34;: &#34;openai&#34;, &#34;model_id&#34;: &#34;text-embedding-3-small&#34;, &#34;api_key&#34;: &#34;sk-xxxxxxxx&#34; } } 配置参数 #     参数 是否必填 描述     text_field 必填 包含待向量化文本的源字段   vector_field 必填 存储生成的向量的目标字段。该字段应映射为 knn_vector 类型   url 必填 Embedding 模型服务的 API 端点 URL   vendor 必填 模型提供商标识。openai 表示 OpenAI 兼容接口，其他值（如 ollama）使用 Ollama 兼容接口   model_id 必填 使用的 Embedding 模型 ID   api_key 可选 API 密钥（使用 OpenAI 兼容接口时通常必填）。存储时会自动加密   dimensions 可选 期望的向量维度。未指定时使用模型默认维度   ignore_missing 可选 为 true 时，源字段缺失则跳过处理。默认为 false   batch_size 可选 批量处理时每次 API 调用包含的文档数。默认为 1   description 可选 处理器的简要描述   if 可选 处理器运行的条件   ignore_failure 可选 为 true 时，处理器出错后忽略继续执行。默认为 false   on_failure 可选 处理器失败时运行的处理器列表   tag 可选 处理器的标识标签    支持的模型服务 #     接口类型 vendor 值 认证方式 典型服务     OpenAI 兼容 openai Authorization: Bearer &lt;api_key&gt; 请求头 OpenAI、阿里云 DashScope、Azure OpenAI、DeepSeek 等   Ollama 兼容 其他任意值（如 ollama） 无认证 Ollama 本地部署    如何使用 #  步骤 1：创建向量索引 #  首先创建包含 KNN 向量字段的索引："
---


# 文本向量化处理器

> **需要 AI 插件和 KNN 插件**

`text_embedding` 处理器在文档写入时自动调用外部 Embedding 模型服务，将文本字段转换为向量并存储到指定的向量字段中，实现"写入即向量化"。该处理器是构建语义搜索和混合搜索的基础组件。

## 语法

```json
{
  "text_embedding": {
    "text_field": "content",
    "vector_field": "content_vector",
    "url": "https://api.openai.com/v1/embeddings",
    "vendor": "openai",
    "model_id": "text-embedding-3-small",
    "api_key": "sk-xxxxxxxx"
  }
}
```

## 配置参数

| 参数 | 是否必填 | 描述 |
|------|---------|------|
| `text_field` | 必填 | 包含待向量化文本的源字段 |
| `vector_field` | 必填 | 存储生成的向量的目标字段。该字段应映射为 `knn_vector` 类型 |
| `url` | 必填 | Embedding 模型服务的 API 端点 URL |
| `vendor` | 必填 | 模型提供商标识。`openai` 表示 OpenAI 兼容接口，其他值（如 `ollama`）使用 Ollama 兼容接口 |
| `model_id` | 必填 | 使用的 Embedding 模型 ID |
| `api_key` | 可选 | API 密钥（使用 OpenAI 兼容接口时通常必填）。存储时会自动加密 |
| `dimensions` | 可选 | 期望的向量维度。未指定时使用模型默认维度 |
| `ignore_missing` | 可选 | 为 `true` 时，源字段缺失则跳过处理。默认为 `false` |
| `batch_size` | 可选 | 批量处理时每次 API 调用包含的文档数。默认为 `1` |
| `description` | 可选 | 处理器的简要描述 |
| `if` | 可选 | 处理器运行的条件 |
| `ignore_failure` | 可选 | 为 `true` 时，处理器出错后忽略继续执行。默认为 `false` |
| `on_failure` | 可选 | 处理器失败时运行的处理器列表 |
| `tag` | 可选 | 处理器的标识标签 |

## 支持的模型服务

| 接口类型 | `vendor` 值 | 认证方式 | 典型服务 |
|---------|------------|---------|---------|
| OpenAI 兼容 | `openai` | `Authorization: Bearer <api_key>` 请求头 | OpenAI、阿里云 DashScope、Azure OpenAI、DeepSeek 等 |
| Ollama 兼容 | 其他任意值（如 `ollama`） | 无认证 | Ollama 本地部署 |

## 如何使用

### 步骤 1：创建向量索引

首先创建包含 KNN 向量字段的索引：

```json
PUT /my_index
{
  "settings": {
    "index.knn": true
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text"
      },
      "content_vector": {
        "type": "knn_vector",
        "dimension": 1536
      }
    }
  }
}
```

### 步骤 2：创建管道

使用 OpenAI 兼容接口创建向量化管道：

```json
PUT /_ingest/pipeline/embedding_pipeline
{
  "description": "写入时自动向量化",
  "processors": [
    {
      "text_embedding": {
        "text_field": "content",
        "vector_field": "content_vector",
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1/embeddings",
        "vendor": "openai",
        "model_id": "text-embedding-v3",
        "api_key": "sk-xxxxxxxx"
      }
    }
  ]
}
```

### 步骤 3：写入数据

```json
PUT /my_index/_doc/1?pipeline=embedding_pipeline
{
  "content": "Easysearch 是一款高性能的搜索引擎"
}
```

处理器会自动将 `content` 字段的文本发送到 Embedding 服务，并将返回的向量存储到 `content_vector` 字段。

### 步骤 4：验证结果

```json
GET /my_index/_doc/1
```

```json
{
  "_source": {
    "content": "Easysearch 是一款高性能的搜索引擎",
    "content_vector": [0.0123, -0.0456, 0.0789, ...]
  }
}
```

### 使用 Ollama 本地模型

```json
PUT /_ingest/pipeline/ollama_embedding_pipeline
{
  "description": "使用 Ollama 本地模型向量化",
  "processors": [
    {
      "text_embedding": {
        "text_field": "content",
        "vector_field": "content_vector",
        "url": "http://localhost:11434/api/embed",
        "vendor": "ollama",
        "model_id": "nomic-embed-text"
      }
    }
  ]
}
```

### 批量写入

使用 `batch_size` 提升批量写入时的向量化效率：

```json
PUT /_ingest/pipeline/batch_embedding_pipeline
{
  "description": "批量向量化",
  "processors": [
    {
      "text_embedding": {
        "text_field": "content",
        "vector_field": "content_vector",
        "url": "https://dashscope.aliyuncs.com/compatible-mode/v1/embeddings",
        "vendor": "openai",
        "model_id": "text-embedding-v3",
        "api_key": "sk-xxxxxxxx",
        "batch_size": 10
      }
    }
  ]
}
```

## 注意事项

- 如果 `vector_field` 已存在于文档中，处理器会抛出异常（设置 `ignore_failure: true` 可跳过）
- API 密钥在管道配置中会被自动加密存储（形如 `ENCRYPTED_VALUE...infinilabs`），查询管道定义时看到的是加密后的值
- 批量写入时，文本为空或缺失的文档会被静默跳过，不会发送到 Embedding 服务
- `vector_field` 的维度必须与所用模型的输出维度一致
- 向量化依赖外部服务，写入速度会受到网络延迟和模型服务吞吐量的影响

## 相关文档

- [写入数据文本向量化]({{< relref "/docs/integrations/ai/ai-api/ingest-text-embedding.md" >}})：完整的端到端工作流指南
- [向量搜索]({{< relref "/docs/features/vector-search/_index.md" >}})：使用向量进行近邻搜索
- [混合搜索]({{< relref "/docs/integrations/ai/ai-api/hybrid-search.md" >}})：结合关键词和向量的混合搜索

