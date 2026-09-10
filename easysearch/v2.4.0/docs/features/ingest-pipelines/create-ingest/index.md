---
title: "创建管道"
date: 0001-01-01
summary: "创建管道 #  使用 PUT _ingest/pipeline API 创建或更新摄取管道。如果指定的 pipeline-id 已存在，则覆盖更新。
请求格式 #  PUT _ingest/pipeline/&lt;pipeline-id&gt; 示例 #  创建一个包含 set 和 uppercase 处理器的管道：
PUT _ingest/pipeline/my-pipeline { &#34;description&#34;: &#34;处理学生数据：设置毕业年份并转大写&#34;, &#34;processors&#34;: [ { &#34;set&#34;: { &#34;description&#34;: &#34;设置毕业年份&#34;, &#34;field&#34;: &#34;grad_year&#34;, &#34;value&#34;: 2023 } }, { &#34;set&#34;: { &#34;description&#34;: &#34;标记已毕业&#34;, &#34;field&#34;: &#34;graduated&#34;, &#34;value&#34;: true } }, { &#34;uppercase&#34;: { &#34;field&#34;: &#34;name&#34; } } ] } 成功响应：
{ &#34;acknowledged&#34;: true } 请求体参数 #     参数 必需 类型 说明     processors 是 array 有序处理器列表，按定义顺序依次执行   description 否 string 管道描述，出现在 GET _ingest/pipeline 的返回中    路径参数 #     参数 必需 类型 说明     pipeline-id 是 string 管道的唯一标识符    查询参数 #     参数 必需 类型 默认值 说明     cluster_manager_timeout 否 时长 30s 等待集群管理器节点响应的超时时间   timeout 否 时长 30s 等待整体响应的超时时间    Mustache 模板 #  处理器参数中可以使用 Mustache 模板引用文档字段。用三个花括号包裹字段名 {{{field_name}}} 可获取未转义的原始值："
---


# 创建管道

使用 `PUT _ingest/pipeline` API 创建或更新摄取管道。如果指定的 `pipeline-id` 已存在，则覆盖更新。

## 请求格式

```
PUT _ingest/pipeline/<pipeline-id>
```

### 示例

创建一个包含 `set` 和 `uppercase` 处理器的管道：

```json
PUT _ingest/pipeline/my-pipeline
{
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
```

成功响应：

```json
{
  "acknowledged": true
}
```

## 请求体参数

| 参数 | 必需 | 类型 | 说明 |
|------|------|------|------|
| `processors` | **是** | array | 有序处理器列表，按定义顺序依次执行 |
| `description` | 否 | string | 管道描述，出现在 `GET _ingest/pipeline` 的返回中 |

## 路径参数

| 参数 | 必需 | 类型 | 说明 |
|------|------|------|------|
| `pipeline-id` | **是** | string | 管道的唯一标识符 |

## 查询参数

| 参数 | 必需 | 类型 | 默认值 | 说明 |
|------|------|------|--------|------|
| `cluster_manager_timeout` | 否 | 时长 | `30s` | 等待集群管理器节点响应的超时时间 |
| `timeout` | 否 | 时长 | `30s` | 等待整体响应的超时时间 |

## Mustache 模板

处理器参数中可以使用 [Mustache](https://mustache.github.io/) 模板引用文档字段。用三个花括号包裹字段名 `{{{field_name}}}` 可获取未转义的原始值：

```json
PUT _ingest/pipeline/dynamic-pipeline
{
  "processors": [
    {
      "set": {
        "field": "{{{role}}}",
        "value": "{{{tenure}}}"
      }
    }
  ]
}
```

此管道会用文档中 `role` 字段的**值**作为目标字段名，用 `tenure` 字段的值作为该字段的值。详见 [在管道中访问数据](./accessing-data.md)。

