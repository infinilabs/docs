---
title: "字段数据参数（Fielddata）"
date: 0001-01-01
summary: "Fielddata 参数 #  fielddata 参数控制 text 字段是否可以用于排序、聚合和脚本。
默认情况下，text 字段不支持排序和聚合——因为 text 字段使用倒排索引，按词项而非完整值存储，无法高效地做正排查找。fielddata 通过将整个倒排索引加载到 JVM 堆内存来实现这一功能，但这 非常消耗内存，通常不推荐使用。
相关指南（先读这些） #    映射基础  映射模式与最佳实践  Doc Values 参数  参数选项 #     值 说明     false 禁用 fielddata。对 text 字段执行排序/聚合会报错。默认值。   true 启用 fielddata。允许对 text 字段排序和聚合，但会大量消耗堆内存。    示例 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;fielddata&#34;: true } } } }  ⚠️ 警告：在生产环境中启用 fielddata 可能导致大量内存消耗，甚至触发断路器（circuit breaker）或 OOM。"
---


# Fielddata 参数

`fielddata` 参数控制 `text` 字段是否可以用于排序、聚合和脚本。

默认情况下，`text` 字段不支持排序和聚合——因为 `text` 字段使用倒排索引，按词项而非完整值存储，无法高效地做正排查找。`fielddata` 通过将整个倒排索引加载到 JVM 堆内存来实现这一功能，但这 **非常消耗内存**，通常不推荐使用。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式与最佳实践]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})
- [Doc Values 参数]({{< relref "doc_values" >}})

## 参数选项

| 值 | 说明 |
|----|------|
| `false` | 禁用 fielddata。对 text 字段执行排序/聚合会报错。**默认值**。 |
| `true` | 启用 fielddata。允许对 text 字段排序和聚合，但会大量消耗堆内存。 |

## 示例

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "fielddata": true
      }
    }
  }
}
```

> ⚠️ **警告**：在生产环境中启用 fielddata 可能导致大量内存消耗，甚至触发断路器（circuit breaker）或 OOM。

## 更好的替代方案

| 需求 | 推荐方案 | 说明 |
|------|----------|------|
| 对字符串做排序/聚合 | 使用 `keyword` 子字段 | `"fields": { "raw": { "type": "keyword" } }` |
| 对字符串做全文搜索 + 聚合 | text + keyword 多字段 | 搜索用 text，聚合用 keyword 子字段 |
| 确实需要对分词后的词项做聚合 | fielddata + fielddata_frequency_filter | 过滤低频词项，减少内存开销 |

## fielddata_frequency_filter

如果确实需要使用 fielddata，可以通过频率过滤减少内存占用：

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "fielddata": true,
        "fielddata_frequency_filter": {
          "min": 0.01,
          "max": 0.5,
          "min_segment_size": 500
        }
      }
    }
  }
}
```

| 参数 | 说明 |
|------|------|
| `min` | 词项最低频率（占比），低于此频率的词项不加载 |
| `max` | 词项最高频率（占比），高于此频率的词项不加载 |
| `min_segment_size` | 分段最小文档数，小于此值的分段不加载 fielddata |

## fielddata 与 doc_values 的对比

| 特性 | fielddata | doc_values |
|------|-----------|------------|
| 适用类型 | `text` 字段 | `keyword`、数值、日期等 |
| 存储位置 | JVM 堆内存（查询时加载） | 磁盘（索引时构建） |
| 内存消耗 | ⚠️ 非常大 | ✅ 极低 |
| 默认状态 | 关闭 | 开启 |
| 推荐度 | ❌ 不推荐 | ✅ 推荐 |

