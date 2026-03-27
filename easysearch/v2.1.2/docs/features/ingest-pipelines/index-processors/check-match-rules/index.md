---
title: "规则匹配处理器"
date: 0001-01-01
summary: "规则匹配处理器 #   需要 Rules 插件和有效许可证
 check_match_rules 处理器将 规则引擎接入 Ingest Pipeline，在文档写入阶段执行规则匹配并自动打标。
语法 #  { &#34;check_match_rules&#34;: { &#34;id&#34;: &#34;my_ruleset&#34; } } 配置参数 #     参数 是否必填 描述     id 必填 规则库 ID（对应已编译的 repo_id）   target_field 可选 匹配结果写入字段，默认 tags   fields 可选 文档字段白名单；指定后仅这些字段按原字段名参与匹配，其余字段内容汇总到 default_match_field   default_match_field 可选 当配置了 fields 时，未包含字段会汇总到该字段名参与匹配，默认 content   regex_start_at_word 可选 正则匹配是否从词边界开始，默认 true   ignore_missing 可选 为 true 时，处理过程中遇到错误会跳过，默认 false   description 可选 处理器描述   if 可选 处理器执行条件   ignore_failure 可选 为 true 时，处理器失败后忽略并继续执行，默认 false   on_failure 可选 处理器失败时执行的处理器列表   tag 可选 处理器标识    工作原理 #   提取文档字段：  未配置 fields：提取所有非 _ 前缀字段（含嵌套展平） 配置 fields：白名单字段保留原名，其余字段值拼接到 default_match_field   执行规则匹配：传入底层规则匹配引擎 写入结果：命中后将标签数组写入 target_field  使用步骤 #  前置条件 #  先导入并编译规则库，详见 规则引擎文档："
---


# 规则匹配处理器

> 需要 Rules 插件和有效许可证

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
| `fields` | 可选 | 文档字段白名单；指定后仅这些字段按原字段名参与匹配，其余字段内容汇总到 `default_match_field` |
| `default_match_field` | 可选 | 当配置了 `fields` 时，未包含字段会汇总到该字段名参与匹配，默认 `content` |
| `regex_start_at_word` | 可选 | 正则匹配是否从词边界开始，默认 `true` |
| `ignore_missing` | 可选 | 为 `true` 时，处理过程中遇到错误会跳过，默认 `false` |
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

先导入并编译规则库，详见 [规则引擎文档]({{< relref "/docs/features/rule-engine.md" >}})：

```json
POST /_match_rules/{repo_id}/_compile
{}
```

### 步骤 1：创建管道

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

### 步骤 2：写入测试数据

```json
PUT /my_index/_doc/1?pipeline=rules_pipeline
{
  "title": "测试文档",
  "content": "这是需要进行规则检测的文本内容"
}
```

### 步骤 3：查看结果

```json
{
  "_source": {
    "title": "测试文档",
    "content": "这是需要进行规则检测的文本内容",
    "tags": ["敏感词规则A", "内容分类规则B"]
  }
}
```

### 指定匹配字段

```json
{
  "check_match_rules": {
    "id": "my_ruleset",
    "fields": ["title", "content"],
    "default_match_field": "content"
  }
}
```

## 注意事项

- 规则库需先通过 `POST /_match_rules/{repo_id}/_compile` 编译
- 正则类型的字段值（以 `(`、`{` 或 `[` 开头）会被识别为正则模式
- 建议将规则库维护、导入、编译流程统一在 rules 文档中管理

