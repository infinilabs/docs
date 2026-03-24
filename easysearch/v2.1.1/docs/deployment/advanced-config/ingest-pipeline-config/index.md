---
title: "Ingest Pipeline 配置"
date: 0001-01-01
summary: "Ingest Pipeline 配置指南 #  Ingest Pipeline 允许在文档索引之前进行预处理（如字段提取、数据转换、富化等）。本文介绍 Ingest 节点的配置和 Pipeline 的高级使用技巧。
 启用 Ingest 功能 #  默认情况下所有节点都具备 Ingest 功能。如需将 Ingest 角色分配给专用节点，请参考 集群节点配置 中的节点角色说明。
 Pipeline 管理 #  创建 Pipeline #  PUT /_ingest/pipeline/my-pipeline { &#34;description&#34;: &#34;日志预处理流水线&#34;, &#34;processors&#34;: [ { &#34;grok&#34;: { &#34;field&#34;: &#34;message&#34;, &#34;patterns&#34;: [&#34;%{COMBINEDAPACHELOG}&#34;] } }, { &#34;date&#34;: { &#34;field&#34;: &#34;timestamp&#34;, &#34;formats&#34;: [&#34;dd/MMM/yyyy:HH:mm:ss Z&#34;] } }, { &#34;remove&#34;: { &#34;field&#34;: &#34;message&#34; } } ] } 查看 Pipeline #  # 查看所有 GET /_ingest/pipeline # 查看指定 GET /_ingest/pipeline/my-pipeline # 查看 Pipeline 统计 GET /_nodes/stats/ingest 删除 Pipeline #  DELETE /_ingest/pipeline/my-pipeline 模拟测试 #  在正式使用前模拟验证 Pipeline 处理效果："
---


# Ingest Pipeline 配置指南

Ingest Pipeline 允许在文档索引之前进行预处理（如字段提取、数据转换、富化等）。本文介绍 Ingest 节点的配置和 Pipeline 的高级使用技巧。

---

## 启用 Ingest 功能

默认情况下所有节点都具备 Ingest 功能。如需将 Ingest 角色分配给专用节点，请参考 [集群节点配置]({{< relref "../config/node-settings/cluster-node.md" >}}) 中的节点角色说明。

---

## Pipeline 管理

### 创建 Pipeline

```bash
PUT /_ingest/pipeline/my-pipeline
{
  "description": "日志预处理流水线",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{COMBINEDAPACHELOG}"]
      }
    },
    {
      "date": {
        "field": "timestamp",
        "formats": ["dd/MMM/yyyy:HH:mm:ss Z"]
      }
    },
    {
      "remove": {
        "field": "message"
      }
    }
  ]
}
```

### 查看 Pipeline

```bash
# 查看所有
GET /_ingest/pipeline

# 查看指定
GET /_ingest/pipeline/my-pipeline

# 查看 Pipeline 统计
GET /_nodes/stats/ingest
```

### 删除 Pipeline

```bash
DELETE /_ingest/pipeline/my-pipeline
```

### 模拟测试

在正式使用前模拟验证 Pipeline 处理效果：

```bash
POST /_ingest/pipeline/my-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "192.168.1.1 - - [28/Jun/2024:10:00:00 +0800] \"GET /index.html HTTP/1.1\" 200 1234"
      }
    }
  ]
}
```

---

## 索引级别 Pipeline 配置

### index.default_pipeline

| 项目 | 说明 |
|------|------|
| **参数** | `index.default_pipeline` |
| **默认值** | 无 |
| **属性** | 动态 |
| **说明** | 索引的默认 Pipeline。写入请求未指定 Pipeline 时使用此配置。设为 `_none` 可禁用 |

```bash
PUT /my-index/_settings
{
  "index.default_pipeline": "my-pipeline"
}
```

### index.final_pipeline

| 项目 | 说明 |
|------|------|
| **参数** | `index.final_pipeline` |
| **默认值** | 无 |
| **属性** | 动态 |
| **说明** | 在默认/请求指定的 Pipeline 之后执行的 Pipeline。无法被请求覆盖，适用于强制审计、字段标准化等场景 |

```bash
PUT /my-index/_settings
{
  "index.final_pipeline": "audit-pipeline"
}
```

### Pipeline 执行顺序

```
文档写入 → request pipeline → default_pipeline → final_pipeline → 索引
                (显式指定)       (未指定时使用)     (始终执行)
```

> 如果请求中指定了 `pipeline` 参数，则使用请求中的 Pipeline 而非 `default_pipeline`。`final_pipeline` 始终执行。

---

## 常用 Processor 速查

| Processor | 用途 | 典型场景 |
|-----------|------|---------|
| `grok` | 正则提取结构化字段 | 日志解析 |
| `date` | 日期解析 | 时间戳标准化 |
| `convert` | 类型转换 | 字符串→数字 |
| `rename` | 字段重命名 | 字段标准化 |
| `remove` | 删除字段 | 清理原始字段 |
| `set` | 设置/覆盖字段值 | 添加默认值 |
| `script` | 脚本处理 | 复杂计算 |
| `split` | 拆分字段 | CSV→数组 |
| `json` | JSON 解析 | 嵌套 JSON 提取 |
| `lowercase` / `uppercase` | 大小写转换 | 字段标准化 |
| `trim` | 去除空白 | 数据清洗 |
| `gsub` | 正则替换 | 文本清洗 |
| `dissect` | 简单模式提取（比 grok 快） | 格式固定的日志 |
| `foreach` | 遍历数组并处理 | 数组字段批量处理 |
| `pipeline` | 调用其他 Pipeline | Pipeline 复用 |
| `drop` | 丢弃文档 | 条件过滤 |
| `fail` | 强制失败 | 数据验证 |

---

## 错误处理

### on_failure 处理器

为 Processor 或整个 Pipeline 设置错误处理：

```bash
PUT /_ingest/pipeline/safe-pipeline
{
  "description": "带错误处理的 Pipeline",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{COMBINEDAPACHELOG}"],
        "on_failure": [
          {
            "set": {
              "field": "_tags",
              "value": ["_grok_parse_failure"]
            }
          }
        ]
      }
    }
  ],
  "on_failure": [
    {
      "set": {
        "field": "error_info",
        "value": "Pipeline 处理失败: {{_ingest.on_failure_message}}"
      }
    }
  ]
}
```

### ignore_failure

忽略单个 Processor 的错误：

```bash
{
  "convert": {
    "field": "status_code",
    "type": "integer",
    "ignore_failure": true
  }
}
```

---

## 性能调优

### 专用 Ingest 节点

高吞吐场景建议部署专用 Ingest 节点，避免 Pipeline 处理影响搜索和索引性能。具体配置方式参考 [集群节点配置]({{< relref "../config/node-settings/cluster-node.md" >}})。

### Pipeline 性能优化建议

| 优化项 | 建议 |
|-------|------|
| Processor 数量 | 尽量精简，合并可合并的操作 |
| `grok` vs `dissect` | 格式固定时优先用 `dissect`（性能更好） |
| `script` 使用 | 避免复杂脚本，优先使用内置 Processor |
| `on_failure` | 设置错误处理，避免写入阻塞 |
| 条件执行 | 使用 `if` 条件减少不必要的处理 |
| 批量大小 | 配合 `_bulk` API，控制每批文档数量 |

### 条件执行示例

仅在特定条件下执行 Processor，减少开销：

```bash
{
  "uppercase": {
    "field": "env",
    "if": "ctx.env != null && ctx.env.length() > 0"
  }
}
```

---

## 监控

```bash
# 查看 Ingest 节点统计
GET /_nodes/stats/ingest

# 关注指标：
# - ingest.total.count: 处理文档总数
# - ingest.total.failed: 失败数
# - ingest.total.time_in_millis: 总耗时
# - ingest.pipelines.<name>.processors: 各 Processor 耗时
```

---

## 延伸阅读

- [集群节点配置]({{< relref "../config/node-settings/cluster-node.md" >}}) - 节点角色设置
- [高级配置参数]({{< relref "./advanced-settings.md" >}}) - 完整参数参考

