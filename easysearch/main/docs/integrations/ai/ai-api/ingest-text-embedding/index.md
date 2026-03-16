---
title: "写入数据文本向量化"
date: 0001-01-01
summary: "写入数据文本向量化 #  Easysearch 使用 Ingest 管道中的一系列处理器，可以对写入的数据进行处理，并且支持对文本进行向量化。本文档介绍如何在 Easysearch 中使用 text_embedding 处理器对写入数据进行向量化。
相关指南（先读这些） #    向量搜索  向量字段建模  AI 集成  先决条件 #  支持与 OpenAI API 兼容的 embedding 接口，支持 Ollama embedding 接口。
需要安装 Easysearch 的 knn 和 ai 插件。
在生产环境中使用数据采集时，您的集群应至少包含一个节点，且该节点的节点角色权限设置为 ingest 。
创建带有向量字段的索引 #  首先，需要创建一个包含 knn mapping 的索引，text_vector 是存储向量的字段，向量维度是 768。
PUT /my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;text_vector&#34;: { &#34;type&#34;: &#34;knn_dense_float_vector&#34;, &#34;knn&#34;: { &#34;dims&#34;: 768, &#34;model&#34;: &#34;lsh&#34;, &#34;similarity&#34;: &#34;cosine&#34;, &#34;L&#34;: 99, &#34;k&#34;: 1 } } } } } 创建或更新 text_embedding 处理器 #  请求路径："
---


# 写入数据文本向量化

Easysearch 使用 Ingest 管道中的一系列处理器，可以对写入的数据进行处理，并且支持对文本进行向量化。本文档介绍如何在 Easysearch 中使用 `text_embedding` 处理器对写入数据进行向量化。

## 相关指南（先读这些）

- [向量搜索]({{< relref "/docs/features/vector-search/_index.md" >}})
- [向量字段建模]({{< relref "/docs/best-practices/data-modeling/vector-fields.md" >}})
- [AI 集成]({{< relref "/docs/integrations/ai/_index.md" >}})

## 先决条件
支持与 OpenAI API 兼容的 embedding 接口，支持 Ollama embedding 接口。

需要安装 Easysearch 的 `knn` 和 `ai` 插件。

在生产环境中使用数据采集时，您的集群应至少包含一个节点，且该节点的节点角色权限设置为 `ingest` 。


## 创建带有向量字段的索引
首先，需要创建一个包含 knn mapping 的索引，text_vector 是存储向量的字段，向量维度是 768。
```auto
PUT /my-index
{
  "mappings": {
    "properties": {
      "text_vector": {
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

## 创建或更新 `text_embedding` 处理器
请求路径：
```auto
PUT _ingest/pipeline/<pipeline-id>
```
请求示例：
```auto
    PUT _ingest/pipeline/text-embedding-pipeline
    {
      "description": "用于生成文本嵌入向量的管道",
      "processors": [
        {
          "text_embedding": {
            "url": "https://api.openai.com/v1/embeddings",
            "vendor": "openai",
            "api_key": "<api_key>",
            "text_field": "input_text",
            "vector_field": "text_vector",
            "model_id": "text-embedding-3-small",
            "dims": 768,
            "ignore_missing": false,
            "ignore_failure": false
          }
        }
      ]
    }
```
### 请求体字段：

下表列出了用于创建或更新管道的请求体字段。

| 参数                   | 是否必填 | 类型  | 说明                                                                            |
| -------------------- | ---- | --- |-------------------------------------------------------------------------------|
| `processors`         | 必填   | 数组  | 处理器列表，按顺序执行。示例中仅包含 `text_embedding` 处理器。                                      |
| `description`        | 可选   | 字符串 | 管道的描述信息，用于说明用途（如“生成文本嵌入向量”）。                                                  |
| **`url`**            | 必填   | 字符串 | Embedding API 的完整 URL（如 https://api.openai.com/v1/embeddings ）。               |
| **`vendor`**         | 必填   | 字符串 | 服务提供商标识，固定为 `"openai"` 或 `"ollama"`。                                          |
| **`api_key`**        | 必填   | 字符串 | API 密钥，需替换为实际值（如 `"sk-xxx"`）。                                                 |
| **`text_field`**     | 必填   | 字符串 | 输入文本的字段名（如 `"input_text"`），需与索引中的字段匹配。                                        |
| **`vector_field`**   | 必填   | 字符串 | 存储向量的目标字段名（如 `"text_vector"`），必须包含在索引的 knn mapping 里。                         |
| **`model_id`**       | 必填   | 字符串 | 模型名称（如 `"text-embedding-3-small"` 或 `"text-embedding-v3"`）。                   |
| **`dims`**           | 必填   | 整数  | 向量维度（需与所选模型匹配，如 `768` 对应 `text-embedding-3-small`）,并与 mapping 里的 `dims` 保持一致。 |
| **`ignore_missing`** | 可选   | 布尔值 | 若 `text_field` 不存在是否跳过处理（默认 `false`）。                                         |
| **`ignore_failure`** | 可选   | 布尔值 | 若处理失败是否继续执行后续处理器（默认 `false`）。                                                 |

若使用国内大模型，例如阿里云的千问，需替换为<br>
url: https://dashscope.aliyuncs.com/compatible-mode/v1/embeddings
<br>
model_id: text-embedding-v3
<br>
dims: 根据模型确定

### 路径参数：

参数 | 是否必填 | 类型  | 说明
:--- |:-----|:----| :---
`pipeline-id` | 必填   | 字符串 | 分配给管道的唯一标识符，即管道 ID。

若已部署 Ollama 服务，按下面示例使用：
```auto
PUT _ingest/pipeline/ollama-embedding-pipeline
{
  "description": "Ollama embedding 示例",
  "processors": [
    {
      "text_embedding": {
        "url": "http://localhost:11434/api/embed",
        "vendor": "ollama",
        "text_field": "input_text",
        "vector_field": "text_vector",
        "model_id": "nomic-embed-text:latest",
        "ignore_missing": false,
        "ignore_failure": false
      }
    }
  ]
}
```

## 查看 `text_embedding` 处理器
查看 `text_embedding` 处理器与查看其他处理器的 api 一致：
```auto
GET _ingest/pipeline
```
返回输出：
```auto
{
  "ollama-embedding-pipeline": {
    "description": "Ollama embedding 示例",
    "processors": [
      {
        "text_embedding": {
          "ignore_failure": false,
          "vendor": "ollama",
          "vector_field": "text_vector",
          "text_field": "input_text",
          "ignore_missing": false,
          "model_id": "nomic-embed-text:latest",
          "url": "http://localhost:11434/api/embed"
        }
      }
    ]
  },
  "text-embedding-pipeline": {
    "description": "用于生成文本嵌入向量的管道",
    "processors": [
      {
        "text_embedding": {
          "ignore_failure": false,
          "dims": 768,
          "api_key": "*************************************************vE",
          "vendor": "openai",
          "vector_field": "text_vector",
          "text_field": "input_text",
          "ignore_missing": false,
          "model_id": "text-embedding-3-small",
          "url": "https://poloai.top/v1/embeddings"
        }
      }
    ]
  }
}
```
`api_key` 会被遮掩，防止泄露

## 使用 `text_embedding` 处理器进行写入时转换
与使用其他处理器一致,
```auto
POST /_bulk?pipeline=text-embedding-pipeline&pretty&refresh=wait_for
{ "index": { "_index": "my-index" } }
{ "input_text": "第一个批量处理的文本。", "source": "bulk api example 1" }
{ "index": { "_index": "my-index", "_id": "bulk_doc_2" } }
{ "input_text": "第二个批量处理的文本，指定了ID。", "priority": 1 }
{ "index": { "_index": "my-index" } }
{ "input_text": "这是另一示例文本。", "tags": ["bulk", "test"] }

```
查看写入结果
```auto
GET my-index/_search
```
返回输出：
```auto
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "my-index",
        "_type": "_doc",
        "_id": "dd2DAZgB6WXmvHRNYksX",
        "_score": 1,
        "_source": {
          "input_text": "第一个批量处理的文本。",
          "text_vector": [
            0.012978158,
            0.007739224,
            -0.015867598,
            0.005287578,
            ......
```

## 删除 `text_embedding` 处理器
与删除其他处理器一致
```auto
DELETE  _ingest/pipeline/text-embedding-pipeline
```

