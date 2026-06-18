---
title: "文档值参数（Doc Values）"
date: 0001-01-01
summary: "Doc Values 参数 #  默认情况下，Easysearch 会为搜索目的索引大多数字段的字段值。doc_values 参数启用文档到词项的正排查找，用于排序、聚合和脚本等操作。
doc_values 参数接受以下选项：
   选项 描述     true 启用字段的 doc_values。默认值为 true。   false 禁用字段的 doc_values。    相关指南（先读这些） #    映射基础  映射模式  示例：创建启用和禁用 doc_values 的索引 #  以下示例请求创建一个索引，其中一个字段启用 doc_values，另一个字段禁用：
PUT my-index-001 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;status_code&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;session_id&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;doc_values&#34;: false } } } } 工作原理 #  doc_values 是一种列式存储结构，与倒排索引互补："
---


# Doc Values 参数

默认情况下，Easysearch 会为搜索目的索引大多数字段的字段值。`doc_values` 参数启用文档到词项的正排查找，用于排序、聚合和脚本等操作。

`doc_values` 参数接受以下选项：

| 选项    | 描述                                       |
| ------- | ------------------------------------------ |
| `true`  | 启用字段的 `doc_values`。默认值为 `true`。 |
| `false` | 禁用字段的 `doc_values`。                  |

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

## 示例：创建启用和禁用 `doc_values` 的索引

以下示例请求创建一个索引，其中一个字段启用 `doc_values`，另一个字段禁用：

```
PUT my-index-001
{
  "mappings": {
    "properties": {
      "status_code": {
        "type": "keyword"
      },
      "session_id": {
        "type": "keyword",
        "doc_values": false
      }
    }
  }
}
```

## 工作原理

doc_values 是一种列式存储结构，与倒排索引互补：

| 数据结构     | 方向         | 用途                                |
| ------------ | ------------ | ----------------------------------- |
| 倒排索引     | 词项 → 文档  | 搜索：给定词项，找到匹配的文档      |
| doc_values   | 文档 → 值    | 排序、聚合、脚本：给定文档，获取值  |

几乎所有字段类型都默认开启 `doc_values`，**`text` 字段除外**（text 字段使用 `fielddata` 来实现类似功能，但不推荐）。

## 何时禁用 doc_values

禁用 `doc_values` 可以减少磁盘空间和索引时间，但该字段将 **无法用于排序、聚合和脚本**。适合以下场景：

- 字段仅用于过滤（如 `session_id`），不需要排序或聚合
- 日志场景中某些字段只做搜索匹配，不做统计分析

> **注意**：`doc_values` 在索引创建后无法修改。如需更改，必须重新创建索引并 reindex 数据。

## 与 fielddata 的区别

| 特性       | doc_values                | fielddata                          |
| ---------- | ------------------------- | ---------------------------------- |
| 存储位置   | 磁盘（索引时构建）        | 堆内存（查询时按需加载）           |
| 适用类型   | keyword、数值、日期等     | text 字段（不推荐）                |
| 性能       | 磁盘 I/O，内存占用低      | 消耗大量堆内存，可能引发 OOM       |
| 推荐度     | ✅ 推荐                   | ⚠️ 仅在无替代方案时使用             |

