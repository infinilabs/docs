---
title: "Pattern 打标处理器"
date: 0001-01-01
summary: "Pattern 打标处理器 #  pattern_tagger 处理器在写入时为日志文档识别 LogPilot 的日志 Pattern 并打标：将每条日志与指定日志流（stream）的合并 Pattern 库（本流 + 父流 + KB 知识包，与 LogPilot 挖掘/标注使用的完全同一套）进行匹配，把命中的 Pattern 身份、匹配准确度等写入文档字段。未命中的行标记为 unmatched，供 LogPilot 后续挖掘与回填；超长文本标记为 skipped，不参与匹配。
可选开启变量抽取（把日志中的变量值结构化存储为 @pattern_vars），并在此基础上高置信度丢弃原始消息以大幅节省存储——配套的搜索管道 pattern_restore 可在查询时从 Pattern 模板 + 变量还原原始消息。
前置条件 #   已安装 ingest-pattern-tagger 插件； 节点 easysearch.yml 中配置 LogPilot 地址（Pattern 库的唯一来源）：  pattern_tagger.logpilot.url: &#34;http://logpilot:29000&#34; pattern_tagger.cache.refresh_interval: 30s LogPilot 的 API 需要鉴权。在 LogPilot 侧创建一个具备流读权限的 Access Token，存入 Easysearch keystore（安全设置，不能写入 easysearch.yml）：
bin/easysearch-keystore add pattern_tagger.logpilot.api_token 插件将凭据以 X-API-TOKEN 头发送；不会发送 Authorization: Bearer——LogPilot 的安全链先校验 Authorization，Access Token 不是合法 JWT 时请求会被直接拒绝（无回退），两种头不能混用。未配置时为匿名访问，仅适用于未开启鉴权的开发环境。"
---


# Pattern 打标处理器

`pattern_tagger` 处理器在写入时为日志文档识别 [LogPilot](https://infinilabs.com) 的日志 Pattern 并打标：将每条日志与指定日志流（stream）的合并 Pattern 库（本流 + 父流 + KB 知识包，与 LogPilot 挖掘/标注使用的完全同一套）进行匹配，把命中的 Pattern 身份、匹配准确度等写入文档字段。未命中的行标记为 `unmatched`，供 LogPilot 后续挖掘与回填；超长文本标记为 `skipped`，不参与匹配。

可选开启**变量抽取**（把日志中的变量值结构化存储为 `@pattern_vars`），并在此基础上**高置信度丢弃原始消息**以大幅节省存储——配套的搜索管道 [`pattern_restore`](/docs/features/query-dsl/search-pipelines/pattern-restore-processor/) 可在查询时从 Pattern 模板 + 变量还原原始消息。

## 前置条件

- 已安装 `ingest-pattern-tagger` 插件；
- 节点 `easysearch.yml` 中配置 LogPilot 地址（Pattern 库的唯一来源）：

```yaml
pattern_tagger.logpilot.url: "http://logpilot:29000"
pattern_tagger.cache.refresh_interval: 30s
```

LogPilot 的 API 需要鉴权。在 LogPilot 侧创建一个具备流读权限的 Access Token，存入 Easysearch keystore（安全设置，不能写入 `easysearch.yml`）：

```bash
bin/easysearch-keystore add pattern_tagger.logpilot.api_token
```

插件将凭据以 `X-API-TOKEN` 头发送；**不会**发送 `Authorization: Bearer`——LogPilot 的安全链先校验 `Authorization`，Access Token 不是合法 JWT 时请求会被直接拒绝（无回退），两种头不能混用。未配置时为匿名访问，仅适用于未开启鉴权的开发环境。

插件内置三级 Pattern 缓存：内存快照（匹配只读它，零锁）→ 本地隐藏索引 `.pattern-tagger-cache`（重启秒级恢复，不依赖 LogPilot）→ LogPilot 定时拉取。LogPilot 不可达时用旧快照继续打标，不会阻塞写入。

## 语法

```
{
  "pattern_tagger": {
    "field": "message",
    "stream_id": "app-logs",
    "max_distance": 0.6
  }
}
```

## 配置参数

| 参数 | 是否必填 | 描述 |
| ---- | -------- | ---- |
| `field` | 可选 | 要分析的日志字段，支持点路径。默认 `message`。 |
| `stream_id` | 必填* | 匹配哪个日志流的合并 Pattern 库，见下方说明。 |
| `max_distance` | 可选 | 匹配距离阈值（0=完全一致，越小越严格）。默认 `0.6`，与 LogPilot 标注一致。 |
| `max_field_length` | 可选 | 长文本防护：字段超过该字符数（默认 `10000`，对齐 LogPilot 挖掘截断长度）则跳过匹配并标记 `@pattern_status: skipped`；`0` 关闭防护。 |
| `target_pattern_id` | 可选 | Pattern ID 落盘字段，默认 `@pattern_id`；设为 `""` 可关闭。 |
| `target_pattern_hash` | 可选 | Pattern 结构哈希落盘字段，默认 `@pattern_hash`；设为 `""` 可关闭。 |
| `target_pattern_distance` | 可选 | 匹配距离（准确度，0=完美）落盘字段，默认 `@pattern_distance`。 |
| `target_pattern_score` | 可选 | 匹配得分（=1-distance，1=完美）落盘字段，默认 `@pattern_score`。 |
| `target_pattern_severity` | 可选 | 命中 Pattern 的严重度落盘字段，默认 `@pattern_severity`；设为 `""` 可关闭。 |
| `target_pattern_status` | 可选 | 匹配状态落盘字段，默认 `@pattern_status`，取值 `matched` / `unmatched` / `skipped`。 |
| `variables` | 可选 | 变量抽取配置，见[变量抽取与丢弃原始消息](#变量抽取与丢弃原始消息)。 |
| `raw_message` | 可选 | 原始消息丢弃策略，见[变量抽取与丢弃原始消息](#变量抽取与丢弃原始消息)。 |
| `patterns` | 可选 | 内联 Pattern 集（离线测试用，绕过 LogPilot 缓存）。 |
| `ignore_missing` | 可选 | 字段缺失或为 `null` 时是否直接跳过。默认 `true`。 |
| `description` / `if` / `ignore_failure` / `on_failure` / `tag` | 可选 | 通用处理器参数。 |

### `stream_id` 的作用

Pattern 库是**按日志流维度**组织与缓存的。`stream_id` 决定处理器拉取并匹配哪一份合并候选集：

- 插件按 `stream_id` 定期调用 LogPilot `GET /logpilot/streams/{stream_id}/_pattern_set`，得到该流的**合并 Pattern 集**（本流活跃 Pattern + 父流继承 + 关联 KB 知识包，与 LogPilot 界面标注、worker 分发使用的完全同一份数据）；
- 不同管道（或同一管道中不同处理器）可指向不同 `stream_id`，各自独立缓存；
- 不配置 `stream_id`（或 LogPilot 未配置）时匹配库为空，所有文档标记 `unmatched`，仅内联 `patterns` 模式可用。

## 写入字段

| 字段 | matched | unmatched | skipped |
| ---- | ------- | --------- | ------- |
| `@pattern_id` | 顶层 Pattern ID（已按层级解析） | `""` | `""` |
| `@pattern_hash` | 命中叶子 Pattern 的结构哈希（确定性、永不过期） | `""` | `""` |
| `@pattern_distance` | 实际距离（0=完美） | `1.0` | `1.0` |
| `@pattern_score` | `1-distance` | `0.0` | `0.0` |
| `@pattern_severity` | 命中 Pattern 的严重度（写入时刻快照） | `""` | `""` |
| `@pattern_status` | `matched` | `unmatched` | `skipped` |
| `@pattern_vars` / `@pattern_extracted` | 开启变量抽取时写入 | - | - |

> 模板（template）**不会**写入文档（长文本纯冗余，可通过 `@pattern_id` / `@pattern_hash` 反查）。严重度（severity）**会**写入：短枚举字典编码后近零存储成本，可对日志索引直接 `@pattern_severity: error` 过滤、按严重度做时序聚合，无需联查 Pattern 库。注意其快照语义——写入的是**摄取时刻** Pattern 库中的严重度，LogPilot 侧后续改级不会自动更新历史文档（需要时可用 `update_by_query` 重跑本管道刷新）。

## 如何使用

### 步骤 1：创建管道

```
PUT _ingest/pipeline/log_pattern_tag
{
  "description": "LogPilot pattern tagging",
  "processors": [
    {
      "pattern_tagger": {
        "field": "message",
        "stream_id": "app-logs",
        "max_distance": 0.6
      }
    }
  ]
}
```

### 步骤 2（可选）：测试管道

生产模式依赖节点已配置 LogPilot 并完成一次缓存拉取。不连 LogPilot 时可用内联 `patterns` 测试：

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "processors": [
      {
        "pattern_tagger": {
          "field": "message",
          "patterns": [
            {
              "pattern_id": "p-uuid-1",
              "pattern_hash": "ph1",
              "tokens": [
                { "type": "static", "value": "ERROR" },
                { "type": "static", "value": "User" },
                { "type": "variable", "value": "word" },
                { "type": "static", "value": "logged" },
                { "type": "static", "value": "in" }
              ]
            }
          ]
        }
      }
    ]
  },
  "docs": [
    { "_source": { "message": "ERROR User alice logged in" } },
    { "_source": { "message": "WARN disk usage above threshold" } }
  ]
}
```

第一条命中（距离 0），第二条未命中：

```
"doc": { "_source": {
  "message": "ERROR User alice logged in",
  "@pattern_id": "p-uuid-1", "@pattern_hash": "ph1",
  "@pattern_distance": 0.0, "@pattern_score": 1.0, "@pattern_status": "matched"
} }
...
"doc": { "_source": {
  "message": "WARN disk usage above threshold",
  "@pattern_id": "", "@pattern_hash": "",
  "@pattern_distance": 1.0, "@pattern_score": 0.0, "@pattern_status": "unmatched"
} }
```

### 步骤 3：摄取文档

```
PUT app-logs/_doc/1?pipeline=log_pattern_tag
{ "message": "ERROR User alice logged in" }
```

或挂为索引默认管道，对单条、批量（`_bulk`）、更新（`_update`）统一生效：

```
PUT app-logs/_settings
{ "index.default_pipeline": "log_pattern_tag" }
```

### 步骤 4：检索与统计

```
GET app-logs/_search
{ "size": 0, "aggs": { "by_pattern": { "terms": { "field": "@pattern_id" } } } }
```

`@pattern_status: unmatched` 的文档可交给 LogPilot 挖掘新 Pattern；`@pattern_distance > 0.5` 的边缘匹配建议优先复查。

## 按条件过滤（可选）

不是所有文档都需要打标。pattern 匹配是 CPU 密集操作，可用处理器级 `if` 条件（所有 ingest 处理器通用的框架能力，详见[控制条件](/docs/features/ingest-pipelines/conditional-execution/)）先把无关日志挡在 `pattern_tagger` 之前，例如只处理 `message` 中包含 `error` 的文档：

```
PUT _ingest/pipeline/log_pattern_tag
{
  "processors": [
    {
      "pattern_tagger": {
        "field": "message",
        "stream_id": "app-logs"
      },
      "if": "ctx.message != null && ctx.message.toLowerCase().contains('error')"
    }
  ]
}
```

更复杂的条件（多字段组合、[正则匹配](/docs/features/ingest-pipelines/conditional-execution/regex-conditionals/)、嵌套字段的空值安全访问 `?.`、`dot_expander` 展开扁平字段等）参见控制条件章节。

注意两点：

- **未过条件的文档完全跳过本处理器**——不匹配、不写任何 `@pattern_*` 字段。这与「参与匹配但未命中」（写 `@pattern_status: unmatched`）语义不同，按 `@pattern_status` 过滤检索时被条件挡掉的文档不会出现，需按需区分。
- `if` 在处理器之前执行，字符串 `contains` 开销极小；先过滤再做 O(C·L) 的模式匹配正是省 CPU 的推荐用法。使用 painless 需节点安装 `lang-painless` 模块（默认发行版自带）。

## 变量抽取与丢弃原始消息

```
PUT _ingest/pipeline/log_pattern_tag
{
  "processors": [
    {
      "pattern_tagger": {
        "field": "message",
        "stream_id": "app-logs",
        "variables": { "enabled": true },
        "raw_message": { "mode": "discard_if_confident", "discard_threshold": 0.1 }
      }
    }
  ]
}
```

- `variables.enabled`：把命中的变量值按位置存入 `@pattern_vars`（如 `["alice", "192.168.1.10", "8080"]`），并写 `@pattern_extracted`；
- `raw_message.mode = discard_if_confident`：仅在 **命中 + 变量抽取成功 + distance ≤ discard_threshold** 三重满足时丢弃原始 `message`（原始行可由 `@pattern_hash` + `@pattern_vars` 还原）；
- 创建管道时会强校验：`discard_if_confident` 不允许关闭 `@pattern_hash` 或 `@pattern_vars` 的写入，否则直接报错；
- 丢弃了原始消息的文档，用 [`pattern_restore`](/docs/features/query-dsl/search-pipelines/pattern-restore-processor/) 搜索管道在查询时还原。

> 注意：完整的"丢弃→还原"闭环要求生产模式（LogPilot + `stream_id`）。内联 `patterns` 仅用于写入侧测试，搜索侧还原依赖共享缓存中的 Pattern 库。

