---
title: "规则匹配处理器"
date: 0001-01-01
summary: "规则匹配处理器 #   需要 Rules 插件、有效 License，以及已成功编译的规则库
 check_match_rules 处理器将 规则引擎接入 Ingest Pipeline，在文档写入阶段执行规则匹配并自动打标。
语法 #  { &#34;check_match_rules&#34;: { &#34;id&#34;: &#34;my_ruleset&#34; } } 配置参数 #     参数 是否必填 描述     id 必填 规则库 ID（对应已编译的 repo_id）   target_field 可选 匹配结果写入字段，默认 tags   fields 可选 文档字段白名单；指定后仅这些字段按原字段名参与匹配，其余字段内容不会丢弃，而是汇总到 default_match_field   default_match_field 可选 当配置了 fields 时，未包含字段会汇总到该字段名参与匹配，默认 content   regex_start_at_word 可选 控制底层正则匹配是否从词边界起算，默认 true，通常保持默认即可   ignore_missing 可选 为 true 时，处理阶段发生异常会直接返回原文档并继续后续处理；默认 false，避免掩盖 License、规则库或配置问题   description 可选 处理器描述   if 可选 处理器执行条件   ignore_failure 可选 为 true 时，处理器失败后忽略并继续执行，默认 false   on_failure 可选 处理器失败时执行的处理器列表   tag 可选 处理器标识    工作原理 #   提取文档字段：  未配置 fields：提取所有非 _ 前缀字段（含嵌套展平） 配置 fields：白名单字段保留原名，其余字段值拼接到 default_match_field   执行规则匹配：传入底层规则匹配引擎 写入结果：命中后将标签数组写入 target_field  使用步骤 #  前置条件 #  在创建 Pipeline 前，请先完成以下检查："
---


# 规则匹配处理器

> 需要 Rules 插件、有效 License，以及已成功编译的规则库

`check_match_rules` 处理器将 [规则引擎]({{< relref "/docs/features/rule-engine.md" >}})接入 Ingest Pipeline，在文档写入阶段执行规则匹配并自动打标。

## 语法

```json
{
  "check_match_rules": {
    "id": "my_ruleset"
  }
}
```

## 配置参数

| 参数 | 是否必填 | 描述 |
|------|---------|------|
| `id` | 必填 | 规则库 ID（对应已编译的 `repo_id`） |
| `target_field` | 可选 | 匹配结果写入字段，默认 `tags` |
| `fields` | 可选 | 文档字段白名单；指定后仅这些字段按原字段名参与匹配，其余字段内容不会丢弃，而是汇总到 `default_match_field` |
| `default_match_field` | 可选 | 当配置了 `fields` 时，未包含字段会汇总到该字段名参与匹配，默认 `content` |
| `regex_start_at_word` | 可选 | 控制底层正则匹配是否从词边界起算，默认 `true`，通常保持默认即可 |
| `ignore_missing` | 可选 | 为 `true` 时，处理阶段发生异常会直接返回原文档并继续后续处理；默认 `false`，避免掩盖 License、规则库或配置问题 |
| `description` | 可选 | 处理器描述 |
| `if` | 可选 | 处理器执行条件 |
| `ignore_failure` | 可选 | 为 `true` 时，处理器失败后忽略并继续执行，默认 `false` |
| `on_failure` | 可选 | 处理器失败时执行的处理器列表 |
| `tag` | 可选 | 处理器标识 |

## 工作原理

1. 提取文档字段：
   - 未配置 `fields`：提取所有非 `_` 前缀字段（含嵌套展平）
   - 配置 `fields`：白名单字段保留原名，其余字段值拼接到 `default_match_field`
2. 执行规则匹配：传入底层规则匹配引擎
3. 写入结果：命中后将标签数组写入 `target_field`

## 使用步骤

### 前置条件

在创建 Pipeline 前，请先完成以下检查：

- 已安装并加载 `rules` 插件
- 已安装有效 License，且 `rule-engine` 特性已启用
- 目标规则库已导入并执行过 `POST /_match_rules/{repo_id}/_compile`

完整安装与 License 步骤见 [规则引擎文档]({{< relref "/docs/features/rule-engine.md" >}})。

最小编译请求示例：

```json
POST /_match_rules/{repo_id}/_compile
{}
```

### 步骤 1：如果只想先验证规则命中，可先直连规则库模拟

在创建 Pipeline 前，如果你只是想确认“这批文档会命中哪些规则”，可以先调用 Rules 插件自己的模拟接口：

```json
POST /_match_rules/my_ruleset/_simulate
{
  "docs": [
    {
      "_id": "doc-1",
      "_source": {
        "title": "测试文档",
        "content": "这是需要进行规则检测的文本内容"
      }
    }
  ]
}
```

这个接口只返回规则命中结果，不会执行完整 ingest pipeline。

注意：

- 它默认按“编译后的规则库 + 输入 `_source`”直接匹配
- 如果你希望匹配语义尽量贴近当前处理器，也可以在请求体中显式传 `fields`、`default_match_field`、`regex_start_at_word`
- 如果规则库在 `_compile` 时声明了 `composite`，这里会直接使用那套已编译结果；但 `/_simulate` 请求本身不支持临时传 `composite`
- 如果你要验证完整 ingest 流程，而不只是规则命中，请继续使用下面的 pipeline `/_simulate` 做最终验证

完整用法见 [规则引擎文档]({{< relref "/docs/features/rule-engine.md" >}})。

### 步骤 2：创建管道

```json
PUT /_ingest/pipeline/rules_pipeline
{
  "description": "实时规则匹配",
  "processors": [
    {
      "check_match_rules": {
        "id": "my_ruleset",
        "target_field": "tags"
      }
    }
  ]
}
```

### 步骤 3：先用 `_simulate` 验证

```json
POST /_ingest/pipeline/rules_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "title": "测试文档",
        "content": "这是需要进行规则检测的文本内容"
      }
    }
  ]
}
```

### 步骤 4：查看模拟结果

```json
{
  "docs": [
    {
      "doc": {
        "_source": {
          "title": "测试文档",
          "content": "这是需要进行规则检测的文本内容",
          "tags": ["敏感词规则A", "内容分类规则B"]
        }
      }
    }
  ]
}
```

### 步骤 5：写入实际数据

```json
PUT /my_index/_doc/1?pipeline=rules_pipeline
{
  "title": "测试文档",
  "content": "这是需要进行规则检测的文本内容"
}
```

### 步骤 6：查看写入结果

```json
{
  "_source": {
    "title": "测试文档",
    "content": "这是需要进行规则检测的文本内容",
    "tags": ["敏感词规则A", "内容分类规则B"]
  }
}
```

## 字段行为

### 默认行为

不配置 `fields` 时，处理器会提取所有非 `_` 前缀字段参与匹配，嵌套对象会按路径展平。

### 指定字段白名单

配置 `fields` 后：

- 白名单中的字段按原字段名参与匹配
- 不在白名单中的字段不会被丢弃
- 这些字段的值会被拼接到 `default_match_field`

示例：

```json
{
  "check_match_rules": {
    "id": "my_ruleset",
    "fields": ["title", "content"],
    "default_match_field": "content"
  }
}
```

如果文档为：

```json
{
  "title": "新品发布",
  "content": "这是正文",
  "author": "张三",
  "source": "新华网"
}
```

且 Pipeline 配置为：

```json
{
  "check_match_rules": {
    "id": "my_ruleset",
    "fields": ["title", "content"],
    "default_match_field": "content"
  }
}
```

那么：

- `title` 仍以 `title` 字段参与匹配
- `content` 仍以 `content` 字段参与匹配
- `author` 和 `source` 的值会追加进 `content`

## 注意事项

- 规则库需先通过 `POST /_match_rules/{repo_id}/_compile` 编译
- `id` 对应的是已经编译成功并同步到当前 Ingest 节点的规则库
- 正则类型的字段值（以 `(`、`{` 或 `[` 开头）会被识别为正则模式
- `ignore_missing = true` 会吞掉处理阶段异常并返回原文档，排障阶段建议保持默认值 `false`
- 如果处理器创建时报 `repository directory does not exist`，通常表示规则库尚未编译到当前节点，或节点同步尚未完成
- 建议将规则库维护、导入、编译流程统一在 [规则引擎文档]({{< relref "/docs/features/rule-engine.md" >}}) 中管理

