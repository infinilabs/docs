---
title: "脚本"
date: 0001-01-01
summary: "脚本 #  Easysearch 提供了内置的脚本语言，可以在查询、聚合、摄入管道、更新操作等多种场景中使用脚本来实现自定义逻辑。
支持的脚本语言 #     语言 用途 沙盒安全 性能      Painless 通用脚本语言，推荐默认使用 ✅ 是 高（编译为字节码）    Mustache 搜索模板（参数化查询） ✅ 是 高    Expressions 高性能数学表达式（排序和评分） ✅ 是 极高（编译为字节码）    脚本使用方式 #  Easysearch 中使用脚本有三种方式：
1. 内联脚本（Inline） #  直接在请求体中编写脚本代码，适合临时性操作：
GET my_index/_search { &#34;script_fields&#34;: { &#34;total_price&#34;: { &#34;script&#34;: { &#34;lang&#34;: &#34;painless&#34;, &#34;source&#34;: &#34;doc[&#39;price&#39;].value * doc[&#39;quantity&#39;].value&#34; } } } } 2."
---


# 脚本

Easysearch 提供了内置的脚本语言，可以在查询、聚合、摄入管道、更新操作等多种场景中使用脚本来实现自定义逻辑。

## 支持的脚本语言

| 语言 | 用途 | 沙盒安全 | 性能 |
|------|------|---------|------|
| [Painless]({{< relref "painless" >}}) | 通用脚本语言，推荐默认使用 | ✅ 是 | 高（编译为字节码） |
| [Mustache]({{< relref "/docs/features/query-dsl/search-template" >}}) | 搜索模板（参数化查询） | ✅ 是 | 高 |
| [Expressions]({{< relref "expressions" >}}) | 高性能数学表达式（排序和评分） | ✅ 是 | 极高（编译为字节码） |

## 脚本使用方式

Easysearch 中使用脚本有三种方式：

### 1. 内联脚本（Inline）

直接在请求体中编写脚本代码，适合临时性操作：

```json
GET my_index/_search
{
  "script_fields": {
    "total_price": {
      "script": {
        "lang": "painless",
        "source": "doc['price'].value * doc['quantity'].value"
      }
    }
  }
}
```

### 2. 存储脚本（Stored Scripts）

将常用脚本预先存储在集群中，通过 ID 引用，适合反复使用的脚本。详见 [存储脚本]({{< relref "stored-scripts" >}})。

```json
POST _scripts/calculate_total
{
  "script": {
    "lang": "painless",
    "source": "doc['price'].value * params['tax_rate']"
  }
}
```

使用时通过 ID 引用：

```json
GET my_index/_search
{
  "script_fields": {
    "total": {
      "script": {
        "id": "calculate_total",
        "params": {
          "tax_rate": 1.08
        }
      }
    }
  }
}
```

### 3. 搜索模板（Search Templates）

使用 Mustache 语言参数化整个查询体，适合面向用户的搜索场景。详见 [Search Template API]({{< relref "/docs/features/query-dsl/search-template" >}})。

## 脚本结构

所有脚本请求都遵循相同的结构：

```json
"script": {
  "lang": "painless",
  "source": "脚本代码",
  "params": {
    "参数名": "参数值"
  }
}
```

| 字段 | 说明 |
|------|------|
| `lang` | 脚本语言，默认为 `painless`，可省略 |
| `source` | 内联脚本的代码内容 |
| `id` | 存储脚本的 ID（与 `source` 二选一） |
| `params` | 传递给脚本的参数（推荐使用参数而非硬编码值，可利用脚本缓存） |

## 脚本可用场景

| 场景 | 说明 | 示例 |
|------|------|------|
| [Script Query]({{< relref "/docs/features/query-dsl/specialized/script" >}}) | 用脚本作为查询条件过滤文档 | 过滤价格 > 阈值的文档 |
| [Script Score]({{< relref "/docs/features/query-dsl/specialized/script-score" >}}) | 用脚本自定义评分逻辑 | 结合时效性和热度的评分公式 |
| [Script Fields]({{< relref "painless-examples" >}}) | 在搜索结果中返回脚本计算的字段 | 计算折扣后价格 |
| [Update API]({{< relref "/docs/features/document-operations/update-api" >}}) | 用脚本更新文档字段 | 增减计数器、追加数组元素 |
| [Update By Query]({{< relref "/docs/features/document-operations/update-by-query" >}}) | 用脚本批量更新文档 | 批量重命名字段 |
| [Ingest Script Processor]({{< relref "/docs/features/ingest-pipelines/index-processors/script" >}}) | 在摄入管道中用脚本处理文档 | 格式转换、字段拆分 |
| [Scripted Metric Aggregation]({{< relref "/docs/features/aggregations/metric-aggregations/scripted-metric" >}}) | 用脚本实现自定义聚合 | 自定义业务指标计算 |
| [Sort Script]({{< relref "/docs/features/query-dsl/sort" >}}) | 用脚本自定义排序逻辑 | 按地理距离和评分综合排序 |
| Reindex Script | 在重建索引时用脚本转换文档 | 字段映射转换 |

## 脚本缓存

Easysearch 会缓存编译后的脚本以提高性能。缓存相关设置：

| 设置 | 默认值 | 说明 |
|------|--------|------|
| `script.cache.max_size` | `3000` | 缓存中最多保留的脚本数 |
| `script.cache.expire` | 无过期 | 缓存过期时间（如 `10m`） |
| `script.max_compilations_rate` | `150/5m` | 每个时间窗口允许的最大编译次数 |

> **最佳实践**：始终使用 `params` 传递变量值，而不是将值硬编码到 `source` 中。这样相同结构的脚本可以共享编译缓存，大幅减少编译开销。

## 脚本安全

Painless 语言在安全沙盒中运行，只允许访问白名单内的 Java API，无法执行文件操作、网络请求或任何危险操作。可以通过以下设置控制脚本功能：

| 设置 | 默认值 | 说明 |
|------|--------|------|
| `script.allowed_types` | 全部 | 允许的脚本类型：`inline`、`stored` |
| `script.allowed_contexts` | 全部 | 允许的脚本上下文（如 `score`、`update`、`ingest`） |

例如，仅允许内联的 Painless 脚本和存储脚本：

```yaml
script.allowed_types: inline, stored
```
