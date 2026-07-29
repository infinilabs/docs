---
title: "获取管道"
date: 0001-01-01
summary: "获取管道 #  使用 GET _ingest/pipeline API 检索管道的定义信息。
请求格式 #  GET _ingest/pipeline/&lt;pipeline-id&gt; 获取所有管道 #  GET _ingest/pipeline 获取指定管道 #  GET _ingest/pipeline/my-pipeline 响应示例：
{ &#34;my-pipeline&#34;: { &#34;description&#34;: &#34;处理学生数据：设置毕业年份并转大写&#34;, &#34;processors&#34;: [ { &#34;set&#34;: { &#34;description&#34;: &#34;设置毕业年份&#34;, &#34;field&#34;: &#34;grad_year&#34;, &#34;value&#34;: 2023 } }, { &#34;set&#34;: { &#34;description&#34;: &#34;标记已毕业&#34;, &#34;field&#34;: &#34;graduated&#34;, &#34;value&#34;: true } }, { &#34;uppercase&#34;: { &#34;field&#34;: &#34;name&#34; } } ] } } 通配符匹配 #  支持通配符查询多个管道：
GET _ingest/pipeline/log-* 查询参数 #     参数 必需 类型 默认值 说明     cluster_manager_timeout 否 时长 30s 等待集群管理器节点响应的超时时间    "
---


# 获取管道

使用 `GET _ingest/pipeline` API 检索管道的定义信息。

## 请求格式

```
GET _ingest/pipeline/<pipeline-id>
```

### 获取所有管道

```
GET _ingest/pipeline
```

### 获取指定管道

```
GET _ingest/pipeline/my-pipeline
```

响应示例：

```json
{
  "my-pipeline": {
    "description": "处理学生数据：设置毕业年份并转大写",
    "processors": [
      {
        "set": {
          "description": "设置毕业年份",
          "field": "grad_year",
          "value": 2023
        }
      },
      {
        "set": {
          "description": "标记已毕业",
          "field": "graduated",
          "value": true
        }
      },
      {
        "uppercase": {
          "field": "name"
        }
      }
    ]
  }
}
```

### 通配符匹配

支持通配符查询多个管道：

```
GET _ingest/pipeline/log-*
```

## 查询参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `cluster_manager_timeout` | 否 | 时长 | `30s` | 等待集群管理器节点响应的超时时间 |

