---
title: "删除管道"
date: 0001-01-01
summary: "删除管道 #  使用 DELETE _ingest/pipeline API 删除摄取管道。
请求格式 #  删除指定管道：
DELETE _ingest/pipeline/&lt;pipeline-id&gt; 使用通配符删除匹配的管道：
DELETE _ingest/pipeline/log-* 删除所有管道（慎用）：
DELETE _ingest/pipeline/* 参数 #     参数 必需 类型 默认值 说明     pipeline-id 是 string — 要删除的管道 ID，支持通配符 *   cluster_manager_timeout 否 时长 30s 等待集群管理器节点响应的超时时间   timeout 否 时长 30s 等待整体响应的超时时间    响应示例 #  { &#34;acknowledged&#34;: true } 注意事项 #   删除一个正在被 index."
---


# 删除管道

使用 `DELETE _ingest/pipeline` API 删除摄取管道。

## 请求格式

删除指定管道：

```
DELETE _ingest/pipeline/<pipeline-id>
```

使用通配符删除匹配的管道：

```
DELETE _ingest/pipeline/log-*
```

删除所有管道（**慎用**）：

```
DELETE _ingest/pipeline/*
```

## 参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `pipeline-id` | **是** | string | — | 要删除的管道 ID，支持通配符 `*` |
| `cluster_manager_timeout` | 否 | 时长 | `30s` | 等待集群管理器节点响应的超时时间 |
| `timeout` | 否 | 时长 | `30s` | 等待整体响应的超时时间 |

## 响应示例

```json
{
  "acknowledged": true
}
```

## 注意事项

- 删除一个正在被 `index.default_pipeline` 或 `index.final_pipeline` 引用的管道，会导致后续写入该索引的操作失败
- 删除操作**不可撤销**。如需备份管道配置，先用 `GET _ingest/pipeline/<pipeline-id>` 获取其定义
- 使用通配符 `*` 删除所有管道时需格外谨慎，建议仅在测试环境中使用

