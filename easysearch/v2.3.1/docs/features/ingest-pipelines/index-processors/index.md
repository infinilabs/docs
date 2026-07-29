---
title: "摄取处理器"
date: 0001-01-01
summary: "摄取处理器 #  摄取处理器是摄取管道的核心组件——每个处理器在文档被索引前执行一个特定的数据转换操作，例如删除字段、从文本中提取值、转换数据格式或补充衍生信息。
查看当前节点可用的处理器列表：
GET /_nodes/ingest?filter_path=nodes.*.ingest.processors 通用参数 #  所有处理器都支持以下通用参数，可与处理器自身参数一起配置：
   参数 类型 默认值 说明     tag string — 为处理器打标签，在 _simulate verbose 输出和节点统计指标中可见   description string — 处理器描述，出现在 _simulate verbose 响应的 description 字段中   if string —  Painless 脚本条件，返回 true 时才执行   ignore_failure boolean false 为 true 时，处理器出错后跳过继续执行后续处理器   on_failure array — 处理器失败后的备用处理器列表，详见 处理管道故障    示例——为 set 处理器添加条件、标签和错误处理："
---


# 摄取处理器

摄取处理器是摄取管道的核心组件——每个处理器在文档被索引前执行一个特定的数据转换操作，例如删除字段、从文本中提取值、转换数据格式或补充衍生信息。

查看当前节点可用的处理器列表：

```
GET /_nodes/ingest?filter_path=nodes.*.ingest.processors
```

## 通用参数

**所有处理器**都支持以下通用参数，可与处理器自身参数一起配置：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `tag` | string | — | 为处理器打标签，在 `_simulate` verbose 输出和节点统计指标中可见 |
| `description` | string | — | 处理器描述，出现在 `_simulate` verbose 响应的 `description` 字段中 |
| `if` | string | — | [Painless 脚本条件](../conditional-execution/)，返回 `true` 时才执行 |
| `ignore_failure` | boolean | `false` | 为 `true` 时，处理器出错后跳过继续执行后续处理器 |
| `on_failure` | array | — | 处理器失败后的备用处理器列表，详见 [处理管道故障](../pipeline-failures.md) |

示例——为 `set` 处理器添加条件、标签和错误处理：

```json
{
  "set": {
    "tag": "set-env",
    "description": "如果缺少 env 字段则设置默认值",
    "if": "ctx.env == null",
    "field": "env",
    "value": "production",
    "ignore_failure": true
  }
}
```

## 支持的处理器

Easysearch 提供 32 个摄取处理器：28 个来自 ingest-common 插件，1 个来自 ingest-user-agent 插件，1 个来自 ingest-geoip 插件，1 个来自 Rules 插件，1 个来自 AI 插件。

| 处理器 | 说明 |
|--------|------|
| [`append`](./append.md) | 向数组字段追加一个或多个值 |
| [`bytes`](./bytes.md) | 将人类可读字节值转换为数值（`"1kb"` → `1024`） |
| [`convert`](./convert.md) | 转换字段数据类型（string → integer 等） |
| [`csv`](./csv.md) | 解析 CSV 行并存储为独立字段 |
| [`date`](./date.md) | 解析日期字符串并设置为文档时间戳 |
| [`date_index_name`](./date-index-name.md) | 根据日期字段将文档路由到时间索引（如 `logs-2025.02.20`） |
| [`dissect`](./dissect.md) | 基于分隔符模式提取结构化字段（比 grok 更快） |
| [`dot_expander`](./dot-expander.md) | 展开带点号的字段名为嵌套对象（`"a.b"` → `{"a":{"b":...}}`） |
| [`drop`](./drop.md) | 丢弃文档，不索引也不报错 |
| [`fail`](./fail.md) | 强制抛出异常并停止管道执行 |
| [`foreach`](./foreach.md) | 对数组字段的每个元素执行指定的处理器 |
| [`geoip`](./geoip.md) | 根据 IP 地址添加地理位置信息（需 ingest-geoip 插件） |
| [`grok`](./grok.md) | 使用正则模式从非结构化文本中提取字段（日志解析利器） |
| [`gsub`](./gsub.md) | 使用正则表达式替换或删除字符串中的子串 |
| [`html_strip`](./html-strip.md) | 移除 HTML 标签，返回纯文本 |
| [`join`](./join.md) | 将数组元素拼接为单个字符串 |
| [`json`](./json.md) | 解析 JSON 字符串为结构化对象 |
| [`kv`](./kv.md) | 解析 `key=value` 对 |
| [`lowercase`](./lowercase.md) | 将字段文本转为小写 |
| [`pipeline`](./pipeline.md) | 调用另一条管道（模块化复用） |
| [`remove`](./remove.md) | 删除一个或多个字段 |
| [`rename`](./rename.md) | 重命名字段 |
| [`script`](./script.md) | 执行 Painless 脚本，可做任意自定义转换 |
| [`set`](./set.md) | 设置字段值（支持 Mustache 模板） |
| [`sort`](./sort.md) | 对数组元素排序（升序/降序） |
| [`split`](./split.md) | 按分隔符将字符串拆分为数组 |
| [`trim`](./trim.md) | 移除字符串字段的首尾空白 |
| [`uppercase`](./uppercase.md) | 将字段文本转为大写 |
| [`urldecode`](./urldecode.md) | 解码 URL 编码的字符串 |
| [`user_agent`](./user-agent.md) | 解析 User-Agent 字符串，提取浏览器、操作系统等信息（需 ingest-user-agent 插件） |
| [`check_match_rules`](./check-match-rules.md) | 实时规则匹配，为命中规则的文档打标签（需 Rules 插件） |
| [`text_embedding`](./text-embedding.md) | 将文本字段转换为向量嵌入（需 AI 插件） |

> **插件处理器**：`geoip`、`user_agent`、`check_match_rules` 和 `text_embedding` 处理器来自独立插件，需安装对应插件后才可使用。
