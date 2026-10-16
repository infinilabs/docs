---
title: "Pattern 还原处理器"
date: 0001-01-01
summary: "Pattern 还原处理器（Pattern Restore Processor） #  pattern_restore 结果增强处理器用于在搜索响应返回时还原原始日志消息：对于被 pattern_tagger 摄取管道以 discard_if_confident 模式丢弃了原始 message 的文档，处理器根据文档上的 @pattern_hash 从 Pattern 缓存中找到对应模板，用 @pattern_vars 中的变量值渲染还原出原始消息，写回目标字段。
处理器全程 best-effort：原始消息仍在、Pattern 已被合并/归档、变量缺失或错位等情况均保持原样返回，不会导致搜索报错。
前置条件 #   已安装 ingest-pattern-tagger 插件（本处理器与其同插件，共享同一份 Pattern 缓存）； 节点已配置 pattern_tagger.logpilot.url 并使用 stream_id 生产模式摄取文档（内联 patterns 仅作用于写入侧，还原依赖共享 Pattern 库）； 摄取管道开启了变量抽取并（可选）丢弃了原始消息。  请求体字段 #  下表列出了该处理器支持的所有配置字段。
   字段 数据类型 说明     stream_id 字符串 从哪个日志流的合并 Pattern 库解析模板，须与摄取管道的 stream_id 一致。必填。   pattern_hash_field 字符串 存放 Pattern 结构哈希的字段名。可选，默认 @pattern_hash。还原按哈希查模板而非 @pattern_id——后者可能已被解析为顶层祖先，模板结构与变量位置不对应。   pattern_vars_field 字符串 存放变量位置数组的字段名。可选，默认 @pattern_vars。   target_field 字符串 还原消息写入的字段名。可选，默认 message。   only_if_missing 布尔值 为 true 时仅在目标字段不存在（原始消息已被丢弃）时才还原，绝不覆盖保留的原始消息。可选，默认 true。   tag 字符串 处理器的唯一标识符。可选。   description 字符串 对该处理器的描述信息。可选。   ignore_failure 布尔值 若为 true，当此处理器执行失败时忽略错误并继续执行管道中的其余处理器。可选，默认值为 false。    示例 #  准备工作 #  创建摄取管道：匹配 app-logs 流的 Pattern，抽取变量并在高置信度时丢弃原始消息："
---


# Pattern 还原处理器（Pattern Restore Processor）

`pattern_restore` 结果增强处理器用于在搜索响应返回时**还原原始日志消息**：对于被 [`pattern_tagger`](/docs/features/ingest-pipelines/index-processors/pattern-tagger/) 摄取管道以 `discard_if_confident` 模式丢弃了原始 `message` 的文档，处理器根据文档上的 `@pattern_hash` 从 Pattern 缓存中找到对应模板，用 `@pattern_vars` 中的变量值渲染还原出原始消息，写回目标字段。

处理器全程 best-effort：原始消息仍在、Pattern 已被合并/归档、变量缺失或错位等情况均保持原样返回，不会导致搜索报错。

## 前置条件

- 已安装 `ingest-pattern-tagger` 插件（本处理器与其同插件，共享同一份 Pattern 缓存）；
- 节点已配置 `pattern_tagger.logpilot.url` 并使用 `stream_id` 生产模式摄取文档（内联 `patterns` 仅作用于写入侧，还原依赖共享 Pattern 库）；
- 摄取管道开启了变量抽取并（可选）丢弃了原始消息。

## 请求体字段

下表列出了该处理器支持的所有配置字段。

| 字段 | 数据类型 | 说明 |
| :--- | :--- | :--- |
| `stream_id` | 字符串 | 从哪个日志流的合并 Pattern 库解析模板，须与摄取管道的 `stream_id` 一致。**必填**。 |
| `pattern_hash_field` | 字符串 | 存放 Pattern 结构哈希的字段名。可选，默认 `@pattern_hash`。还原**按哈希查模板**而非 `@pattern_id`——后者可能已被解析为顶层祖先，模板结构与变量位置不对应。 |
| `pattern_vars_field` | 字符串 | 存放变量位置数组的字段名。可选，默认 `@pattern_vars`。 |
| `target_field` | 字符串 | 还原消息写入的字段名。可选，默认 `message`。 |
| `only_if_missing` | 布尔值 | 为 `true` 时仅在目标字段不存在（原始消息已被丢弃）时才还原，绝不覆盖保留的原始消息。可选，默认 `true`。 |
| `tag` | 字符串 | 处理器的唯一标识符。可选。 |
| `description` | 字符串 | 对该处理器的描述信息。可选。 |
| `ignore_failure` | 布尔值 | 若为 `true`，当此处理器执行失败时忽略错误并继续执行管道中的其余处理器。可选，默认值为 `false`。 |

## 示例

### 准备工作

创建摄取管道：匹配 `app-logs` 流的 Pattern，抽取变量并在高置信度时丢弃原始消息：

```auto
PUT /_ingest/pipeline/log_pattern_tag
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

索引文档（命中已知 Pattern、完全匹配，原始消息被丢弃）：

```auto
POST /app-logs/_doc/1?pipeline=log_pattern_tag
{ "message": "ERROR User alice logged in from 192.168.1.10 port 8080" }
```

落盘后的文档 `_source` 类似如下——`message` 已不存在，只保留 Pattern 身份与变量：

```auto
{
  "@pattern_id": "3f2c8a10-...",
  "@pattern_hash": "p95dmcc",
  "@pattern_status": "matched",
  "@pattern_distance": 0.0,
  "@pattern_score": 1.0,
  "@pattern_extracted": true,
  "@pattern_vars": ["alice", "192.168.1.10", "8080"]
}
```

### 创建搜索管道

```auto
PUT /_search/pipeline/pattern_restore
{
  "enrich_processors": [
    {
      "pattern_restore": {
        "stream_id": "app-logs"
      }
    }
  ]
}
```

### 使用搜索管道

不使用搜索管道时，响应中的文档没有 `message` 字段：

```auto
GET /app-logs/_search
```

<details open markdown="block">
  <summary>
    响应
  </summary>

```auto
{
  "hits": {
    "hits": [
      {
        "_index": "app-logs",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "@pattern_id": "3f2c8a10-...",
          "@pattern_hash": "p95dmcc",
          "@pattern_status": "matched",
          "@pattern_distance": 0.0,
          "@pattern_score": 1.0,
          "@pattern_extracted": true,
          "@pattern_vars": ["alice", "192.168.1.10", "8080"]
        }
      }
    ]
  }
}
```
</details>

通过 `search_pipeline` 查询参数使用管道，`message` 被还原：

```auto
GET /app-logs/_search?search_pipeline=pattern_restore
```

<details open markdown="block">
  <summary>
    响应
  </summary>

```auto
{
  "hits": {
    "hits": [
      {
        "_index": "app-logs",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "message": "ERROR User alice logged in from 192.168.1.10 port 8080",
          "@pattern_id": "3f2c8a10-...",
          "@pattern_hash": "p95dmcc",
          "@pattern_status": "matched",
          "@pattern_distance": 0.0,
          "@pattern_score": 1.0,
          "@pattern_extracted": true,
          "@pattern_vars": ["alice", "192.168.1.10", "8080"]
        }
      }
    ]
  }
}
```
</details>

## 保真度与边界

- **空白规范化**：还原按单空格连接模板片段，原文中 token 间的多空格/制表符不可恢复，还原结果为空白规范化形式（语义无损）。
- **还原依赖 Pattern 存续**：若命中的叶子 Pattern 后续在 LogPilot 中被合并或归档（不再出现在 Pattern 库中），对应旧文档无法还原，处理器会静默跳过。这也是摄取侧 `discard_if_confident` 默认不开启、仅在确认 Pattern 生命周期策略后启用的原因。
- **按哈希而非 ID 还原**：`@pattern_hash` 是命中叶子模板的确定性结构身份，与 `@pattern_vars` 位置严格对齐；`@pattern_id` 可能是解析后的顶层祖先 ID，不用于模板查找。

