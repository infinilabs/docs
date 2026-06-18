---
title: "忽略格式错误参数（Ignore Malformed）"
date: 0001-01-01
summary: "Ignore Malformed 参数 #  ignore_malformed 参数控制在写入格式错误的数据时，是否忽略该值而不是拒绝整个文档。
默认情况下，写入一个类型不匹配的值（如向数值字段写入字符串）会导致整个文档被拒绝。启用 ignore_malformed 后，格式错误的值会被静默忽略，文档的其他字段仍然正常索引。
相关指南（先读这些） #    映射基础  Index API  参数选项 #     值 说明     false 格式错误的值导致整个文档被拒绝。默认值。   true 格式错误的值被忽略，文档其余部分正常索引。    字段级示例 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;price&#34;: { &#34;type&#34;: &#34;integer&#34;, &#34;ignore_malformed&#34;: true }, &#34;name&#34;: { &#34;type&#34;: &#34;keyword&#34; } } } } PUT my-index/_doc/1 { &#34;price&#34;: &#34;not_a_number&#34;, &#34;name&#34;: &#34;测试产品&#34; } 文档 1 会被成功写入："
---


# Ignore Malformed 参数

`ignore_malformed` 参数控制在写入格式错误的数据时，是否忽略该值而不是拒绝整个文档。

默认情况下，写入一个类型不匹配的值（如向数值字段写入字符串）会导致整个文档被拒绝。启用 `ignore_malformed` 后，格式错误的值会被静默忽略，文档的其他字段仍然正常索引。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [Index API]({{< relref "/docs/features/document-operations/index-api" >}})

## 参数选项

| 值 | 说明 |
|----|------|
| `false` | 格式错误的值导致整个文档被拒绝。**默认值**。 |
| `true` | 格式错误的值被忽略，文档其余部分正常索引。 |

## 字段级示例

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "price": {
        "type": "integer",
        "ignore_malformed": true
      },
      "name": {
        "type": "keyword"
      }
    }
  }
}

PUT my-index/_doc/1
{
  "price": "not_a_number",
  "name": "测试产品"
}
```

文档 1 会被成功写入：
- `name` 字段正常索引
- `price` 字段的值 `"not_a_number"` 被忽略（不索引、不存入 doc_values），但仍保留在 `_source` 中

## 索引级默认值

可以在索引 settings 中设置全局默认值：

```json
PUT my-index
{
  "settings": {
    "index.mapping.ignore_malformed": true
  },
  "mappings": {
    "properties": {
      "price": {
        "type": "integer"
      },
      "strict_field": {
        "type": "integer",
        "ignore_malformed": false
      }
    }
  }
}
```

上面的配置中，所有字段默认忽略格式错误（来自 settings），但 `strict_field` 显式覆盖为 `false`（严格模式）。

## 适用场景

| 场景 | 建议 |
|------|------|
| 数据源不可控（如爬虫、第三方系统） | `ignore_malformed: true` |
| 数据源可控、严格校验 | `ignore_malformed: false`（默认） |
| 日志/监控场景，某些字段偶尔格式异常 | `ignore_malformed: true` |
| 金融/订单等关键数据 | `ignore_malformed: false` |

## 被忽略的字段行为

| 行为 | 说明 |
|------|------|
| `_source` | ✅ 保留原始值 |
| 倒排索引 | ❌ 不索引（无法搜索） |
| doc_values | ❌ 不存储（无法排序/聚合） |
| `_ignored` 元数据字段 | ✅ 记录被忽略的字段名 |

可以通过 `_ignored` 字段查找包含被忽略字段的文档：

```json
GET my-index/_search
{
  "query": {
    "exists": { "field": "_ignored" }
  }
}
```

## 注意事项

- `ignore_malformed` 不适用于 `object` 和 `nested` 字段类型（JSON 结构错误总是会报错）
- 被忽略的字段仍占用 `_source` 空间
- 开启此参数可能掩盖数据质量问题，建议配合监控 `_ignored` 字段使用

