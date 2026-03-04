---
title: "文本向量化（已废弃）"
date: 0001-01-01
summary: "文本向量化（已废弃） #   注意：本文档描述的功能已不再支持，将在下个版本删除，请使用新的 写入数据文本向量化替代。
 本文档介绍如何在 Easysearch 中集成和使用预先部署的 Ollama 服务来生成文本嵌入向量。
先决条件 #  需要预先部署好 Ollama 服务，现阶段集成的服务版本是 0.5.4。
可以用以下命令测试服务是否正常：
curl http://localhost:11434/api/embed -d &#39;{ &#34;model&#34;: &#34;nomic-embed-text:latest&#34;, &#34;input&#34;: &#34;Why is the sky blue?&#34; }&#39; 配置 Ollama 服务 #  可以通过 ollama_url 配置项指定 Ollama 服务的地址。您可以通过以下 API 查看当前配置：
GET _cluster/settings?flat_settings=true&amp;include_defaults=true&amp;filter_path=*.ollama_url 如果没有修改，会输出默认值：
{ &#34;defaults&#34;: { &#34;ollama_url&#34;: &#34;http://localhost:11434&#34; } } REST API #  POST /_ai/embed { &#34;model&#34;: &#34;模型名称&#34;, &#34;input&#34;: &#34;文本内容&#34; } 请求示例 #  POST /_ai/embed { &#34;model&#34;: &#34;nomic-embed-text:latest&#34;, &#34;input&#34;: &#34;Llamas are members of the camelid family&#34; } 批量生成 Embeddings #  可以一次为多个文本生成嵌入向量。"
---


# 文本向量化（已废弃）

> **注意**：本文档描述的功能已不再支持，将在下个版本删除，请使用新的[写入数据文本向量化]({{< relref "./ingest-text-embedding" >}})替代。

本文档介绍如何在 Easysearch 中集成和使用预先部署的 Ollama 服务来生成文本嵌入向量。


## 先决条件
需要预先部署好 Ollama 服务，现阶段集成的服务版本是 0.5.4。

可以用以下命令测试服务是否正常：
```auto
curl http://localhost:11434/api/embed -d '{
  "model": "nomic-embed-text:latest",
  "input": "Why is the sky blue?"
}'
```

## 配置 Ollama 服务
可以通过 ollama_url 配置项指定 Ollama 服务的地址。您可以通过以下 API 查看当前配置：

```auto
GET _cluster/settings?flat_settings=true&include_defaults=true&filter_path=*.ollama_url
```
如果没有修改，会输出默认值：
```auto
{
  "defaults": {
    "ollama_url": "http://localhost:11434"
  }
}
```

## REST API
```auto
POST /_ai/embed
{
  "model": "模型名称",
  "input": "文本内容"
}
```

#### 请求示例
```auto
POST /_ai/embed
{
  "model": "nomic-embed-text:latest",
  "input": "Llamas are members of the camelid family"
}
```

#### 批量生成 Embeddings
可以一次为多个文本生成嵌入向量。

请求格式：
```auto
POST /_ai/embed
{
  "model": "模型名称",
  "input": ["文本1", "文本2", ...]
}
```
#### 示例响应
```auto
{
  "embeddings": [
    [
      0.00971285,
      0.04449268,
      -0.14063065,
      0.0013162489,
      ......
    ],
    [
      0.010429025,
      0.014321253,
      -0.12902334,
      -0.03530379,
      0.046050403,
      0.04550423,
      ......
    ]
  ]
}
```
对于批量处理，建议一次发送多个文本而不是多次调用。

如果收到连接错误，请检查 ollama_url 配置是否正确。


