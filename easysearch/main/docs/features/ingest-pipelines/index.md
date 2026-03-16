---
title: "摄取管道"
date: 0001-01-01
description: "在写入链路上做预处理：使用摄取管道对文档进行清洗、拆分与丰富。"
summary: "摄取管道（Ingest Pipelines） #  摄取管道是 Easysearch 的**&ldquo;写入时数据治理&rdquo;**机制——在文档真正落盘前，按顺序经过一组处理器（processors），对文档做清洗、转换、补充字段等预处理。
典型场景：
 上游系统&quot;粗糙地&quot;丢过来的数据，先在管道中做一次结构化/规范化 把一部分计算前移到写入阶段，减轻查询时的实时计算压力 强制注入合规/审计字段（final_pipeline），无法被客户端绕过  基本概念 #  管道（pipeline）由一个有序的处理器列表组成，每个处理器做一件事。文档在被索引前依次经过所有处理器，前一个处理器的输出是后一个处理器的输入。
管道存储在集群状态（cluster state）中，所有节点可见，定义后即可使用。
管道定义结构 #  PUT _ingest/pipeline/my-pipeline { &#34;description&#34;: &#34;添加摄取时间戳，丢弃原始字段&#34;, &#34;processors&#34;: [ { &#34;set&#34;: { &#34;field&#34;: &#34;event.ingested_at&#34;, &#34;value&#34;: &#34;{{_ingest.timestamp}}&#34; } }, { &#34;remove&#34;: { &#34;field&#34;: &#34;raw&#34;, &#34;ignore_failure&#34;: true } } ] }    字段 必需 说明     description 否 管道的描述文本   processors 是 有序处理器列表    处理器通用参数 #  每个处理器除了自身的配置参数外，都支持以下 5 个通用参数："
---


# 摄取管道（Ingest Pipelines）

摄取管道是 Easysearch 的**"写入时数据治理"**机制——在文档真正落盘前，按顺序经过一组处理器（processors），对文档做清洗、转换、补充字段等预处理。

典型场景：

- 上游系统"粗糙地"丢过来的数据，先在管道中做一次结构化/规范化
- 把一部分计算前移到写入阶段，减轻查询时的实时计算压力
- 强制注入合规/审计字段（`final_pipeline`），无法被客户端绕过

## 基本概念

管道（pipeline）由一个有序的处理器列表组成，每个处理器做一件事。文档在被索引前依次经过所有处理器，前一个处理器的输出是后一个处理器的输入。

管道存储在集群状态（cluster state）中，所有节点可见，定义后即可使用。

### 管道定义结构

```json
PUT _ingest/pipeline/my-pipeline
{
  "description": "添加摄取时间戳，丢弃原始字段",
  "processors": [
    {
      "set": {
        "field": "event.ingested_at",
        "value": "{{_ingest.timestamp}}"
      }
    },
    {
      "remove": {
        "field": "raw",
        "ignore_failure": true
      }
    }
  ]
}
```

| 字段 | 必需 | 说明 |
|------|------|------|
| `description` | 否 | 管道的描述文本 |
| `processors` | **是** | 有序处理器列表 |

### 处理器通用参数

每个处理器除了自身的配置参数外，都支持以下 5 个通用参数：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `tag` | string | — | 处理器标签，用于调试和监控指标中的标识 |
| `description` | string | — | 处理器描述，出现在 `_simulate` 的 verbose 输出中 |
| `if` | string | — | Painless 脚本条件，返回 `true` 时才执行该处理器 |
| `ignore_failure` | boolean | `false` | 为 `true` 时，该处理器出错后跳过并继续执行后续处理器 |
| `on_failure` | array | — | 该处理器失败后执行的备用处理器列表 |

> `if` 条件中的文档上下文是**只读**的：尝试在 `if` 脚本中修改文档会抛出 `UnsupportedOperationException`。详见 [条件执行](./conditional-execution/)。

## 管道生命周期 API

管道的增删改查通过 REST API 完成：

| 操作 | API | 详细文档 |
|------|-----|----------|
| 创建/更新 | `PUT _ingest/pipeline/{id}` | [创建管道](./create-ingest.md) |
| 查看 | `GET _ingest/pipeline/{id}` | [获取管道](./get-ingest.md) |
| 删除 | `DELETE _ingest/pipeline/{id}` | [删除管道](./delete-ingest.md) |
| 模拟 | `POST _ingest/pipeline/{id}/_simulate` | [模拟管道](./simulate-ingest.md) |

## 使用管道的三种方式

### 方式一：请求级别指定

在写入请求的 URL 参数中指定 `pipeline`：

```json
POST my-index/_doc?pipeline=my-pipeline
{
  "message": "原始日志内容",
  "raw": "some-raw-field"
}
```

### 方式二：索引默认管道（`default_pipeline`）

在[索引设置]({{< relref "/docs/operations/data-management/index-settings.md" >}})中配置 `index.default_pipeline`，所有写入该索引的文档自动经过该管道。如果请求同时指定了 `pipeline` 参数，则请求级别的管道**会覆盖**默认管道：

```json
PUT /my-index/_settings
{
  "index.default_pipeline": "my-pipeline"
}
```

### 方式三：最终管道（`final_pipeline`）

`index.final_pipeline` 在 `default_pipeline`（或请求级管道）之后执行，且**不可被请求级别的 `pipeline` 参数覆盖**——适合做强制性的合规/审计字段注入：

```json
PUT /my-index/_settings
{
  "index.final_pipeline": "audit-pipeline"
}
```

> 两个设置的默认值都是 `"_none"`（不启用），均支持动态修改。

### 管道执行顺序

```
写入请求到达
    ↓
请求级 pipeline 或 index.default_pipeline
    ↓
index.final_pipeline
    ↓
Lucene 索引写入
```

> `final_pipeline` 中**不允许修改目标索引**（`_index` 字段）。如果尝试修改，会抛出 `IllegalStateException`。

## 调试：_simulate API

在正式使用前，用 `_simulate` API 测试管道效果，不会实际索引文档：

```json
POST _ingest/pipeline/my-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "test message",
        "raw": "should-be-removed"
      }
    }
  ]
}
```

加上 `?verbose` 参数可以看到每个处理器执行后的中间状态，逐步排查问题。详见 [模拟管道](./simulate-ingest.md)。

## 错误处理

处理器失败时的默认行为是中止整条管道，文档不会被索引。两种应对方式：

**跳过错误**：设置 `"ignore_failure": true`，失败的处理器被跳过，后续处理器继续执行。

**备用逻辑**：通过 `on_failure` 指定备用处理器列表，在失败后做补救（如记录错误信息）：

```json
{
  "date": {
    "field": "timestamp_field",
    "formats": ["yyyy-MM-dd HH:mm:ss"],
    "target_field": "@timestamp",
    "on_failure": [
      {
        "set": {
          "field": "ingest_error",
          "value": "date parse failed: {{{_ingest.on_failure_message}}}"
        }
      }
    ]
  }
}
```

详见 [处理管道故障](./pipeline-failures.md)。

## 在管道中访问数据

处理器通过 `ctx` 对象访问文档字段和元数据：

| 访问方式 | 示例 |
|----------|------|
| 文档字段 | `ctx.user.name` |
| 元数据字段 | `ctx._index`、`ctx._id`、`ctx._routing` |
| 摄取时间戳 | `{{_ingest.timestamp}}` |
| Mustache 模板 | `{{{field_name}}}` |

详见 [在管道中访问数据](./accessing-data.md)。

## 处理器速查表

Easysearch 内置 29 个处理器（28 个来自 ingest-common 模块，1 个来自 ingest-user-agent 插件），按功能分组如下：

### 文本处理

| 处理器 | 说明 |
|--------|------|
| [`lowercase`](./index-processors/lowercase.md) | 转为小写 |
| [`uppercase`](./index-processors/uppercase.md) | 转为大写 |
| [`trim`](./index-processors/trim.md) | 去除首尾空白 |
| [`gsub`](./index-processors/gsub.md) | 正则替换 |
| [`html_strip`](./index-processors/html-strip.md) | 去除 HTML 标签 |
| [`urldecode`](./index-processors/urldecode.md) | URL 解码 |

### 解析与提取

| 处理器 | 说明 |
|--------|------|
| [`grok`](./index-processors/grok.md) | 正则模式解析非结构化文本（日志等） |
| [`dissect`](./index-processors/dissect.md) | 基于分隔符的轻量解析（比 grok 更快） |
| [`json`](./index-processors/json.md) | 解析 JSON 字符串为嵌套对象 |
| [`kv`](./index-processors/kv.md) | 解析 `key=value` 对 |
| [`csv`](./index-processors/csv.md) | 解析 CSV 数据 |
| [`user_agent`](./index-processors/user-agent.md) | 解析 User-Agent 字符串（需 ingest-user-agent 插件） |

### 字段操作

| 处理器 | 说明 |
|--------|------|
| [`set`](./index-processors/set.md) | 设置/更新字段值 |
| [`remove`](./index-processors/remove.md) | 删除字段 |
| [`rename`](./index-processors/rename.md) | 重命名字段 |
| [`append`](./index-processors/append.md) | 向数组字段追加值 |
| [`convert`](./index-processors/convert.md) | 转换字段类型（string→int 等） |
| [`dot_expander`](./index-processors/dot-expander.md) | 展开点号字段名为嵌套对象 |
| [`split`](./index-processors/split.md) | 按分隔符拆分为数组 |
| [`join`](./index-processors/join.md) | 数组拼接为字符串 |
| [`sort`](./index-processors/sort.md) | 对数组排序 |
| [`bytes`](./index-processors/bytes.md) | 人类可读字节值转为数值（`"1kb"` → `1024`） |

### 日期处理

| 处理器 | 说明 |
|--------|------|
| [`date`](./index-processors/date.md) | 解析日期字符串 |
| [`date_index_name`](./index-processors/date-index-name.md) | 根据日期字段路由到时间索引 |

### 控制流

| 处理器 | 说明 |
|--------|------|
| [`pipeline`](./index-processors/pipeline.md) | 调用另一条管道（模块化复用） |
| [`foreach`](./index-processors/foreach.md) | 遍历数组，对每个元素执行处理器 |
| [`drop`](./index-processors/drop.md) | 丢弃文档（不索引） |
| [`fail`](./index-processors/fail.md) | 强制失败并返回自定义错误消息 |

### 高级

| 处理器 | 说明 |
|--------|------|
| [`script`](./index-processors/script.md) | 执行 Painless 脚本，可做任意转换 |

详细参数与完整示例见 [摄取处理器参考](./index-processors/)。

## 与整体数据接入方案的关系

摄取管道是写入链路的**最后一道预处理环节**。在生产环境中，通常配合外部采集工具一起使用：

```
上游（Logstash / Filebeat / 自研采集 / 消息队列）
    ↓ HTTP / Bulk API
Easysearch 协调节点接收请求
    ↓ default_pipeline / 请求指定的 pipeline
    ↓ final_pipeline（不可绕过）
主分片执行 Lucene 索引写入
    ↓ 复制到副本分片
```

**设计建议**：

- **轻量清洗**（字段重命名、类型转换、时间戳注入）→ 放在摄取管道
- **重计算逻辑**（多表关联、复杂 ETL、外部 API 调用）→ 放在上游 Logstash 或自研服务
- **管道复用** → 用 `pipeline` 处理器实现模块化，避免重复定义

## 小结

| 要点 | 说明 |
|------|------|
| 定位 | 文档写入前的预处理流水线 |
| 存储 | 集群状态（cluster state），所有节点可见 |
| 处理器 | 29 个内置，覆盖文本、解析、字段操作、日期、控制流 |
| 执行顺序 | `default_pipeline` → `final_pipeline`（不可绕过） |
| 调试 | `_simulate` API，支持 `verbose` 模式逐步追踪 |
| 错误处理 | `ignore_failure` 跳过、`on_failure` 备用逻辑 |

## 下一步

- [创建管道](./create-ingest.md) / [获取管道](./get-ingest.md) / [删除管道](./delete-ingest.md)：管道 CRUD API
- [模拟管道](./simulate-ingest.md)：调试与测试
- [在管道中访问数据](./accessing-data.md)：`ctx`、Mustache 模板、元数据字段
- [条件执行](./conditional-execution/)：`if` 条件、正则匹配、复杂逻辑
- [处理管道故障](./pipeline-failures.md)：`ignore_failure`、`on_failure`、监控指标
- [摄取处理器参考](./index-processors/)：每个处理器的详细参数与示例

### 相关内容

- [文档操作]({{< relref "/docs/features/document-operations/_index.md" >}})：文档写入 API 与批量操作
- [索引设置]({{< relref "/docs/operations/data-management/index-settings.md" >}})：`default_pipeline`、`final_pipeline` 等索引级配置
- [Logstash 集成]({{< relref "/docs/integrations/ingest/logstash.md" >}})：与 Logstash 配合使用
- [Filebeat / FluentBit 集成]({{< relref "/docs/integrations/ingest/filebeat-fluentbit.md" >}})：轻量采集器接入

