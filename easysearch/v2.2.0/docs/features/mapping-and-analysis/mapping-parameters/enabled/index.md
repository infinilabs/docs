---
title: "启用参数（Enabled）"
date: 0001-01-01
summary: "Enabled 参数 #  enabled 参数允许您控制 Easysearch 是否解析字段的内容。此参数可以应用于顶级映射定义和对象字段。
enabled 参数接受以下值：
   参数 描述     true 字段被解析和索引。默认值为 true。   false 字段不被解析或索引，但仍可从 _source 字段中检索。当 enabled 设置为 false 时，Easysearch 将字段的值存储在 _source 字段中，但不索引或解析其内容。这对于您想要存储但不需要搜索、排序或聚合的字段很有用。    相关指南（先读这些） #    映射基础  映射模式  示例：使用 enabled 参数 #  在以下示例请求中，session_data 字段被禁用。Easysearch 将其内容存储在 _source 字段中，但不对其进行索引或解析：
PUT my-index-002 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;user_id&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;last_updated&#34;: { &#34;type&#34;: &#34;date&#34; }, &#34;session_data&#34;: { &#34;type&#34;: &#34;object&#34;, &#34;enabled&#34;: false } } } } 使用场景 #  存储原始 JSON 但不索引 #  enabled: false 非常适合那些需要随文档一起返回但不需要被搜索的数据："
---


# Enabled 参数

`enabled` 参数允许您控制 Easysearch 是否解析字段的内容。此参数可以应用于顶级映射定义和对象字段。

`enabled` 参数接受以下值：

| 参数    | 描述                                                                                                                                                                                                        |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `true`  | 字段被解析和索引。默认值为 `true`。                                                                                                                                                                         |
| `false` | 字段不被解析或索引，但仍可从 `_source` 字段中检索。当 `enabled` 设置为 `false` 时，Easysearch 将字段的值存储在 `_source` 字段中，但不索引或解析其内容。这对于您想要存储但不需要搜索、排序或聚合的字段很有用。 |

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

## 示例：使用 `enabled` 参数

在以下示例请求中，`session_data` 字段被禁用。Easysearch 将其内容存储在 `_source` 字段中，但不对其进行索引或解析：

```
PUT my-index-002
{
  "mappings": {
    "properties": {
      "user_id": {
        "type": "keyword"
      },
      "last_updated": {
        "type": "date"
      },
      "session_data": {
        "type": "object",
        "enabled": false
      }
    }
  }
}
```

## 使用场景

### 存储原始 JSON 但不索引

`enabled: false` 非常适合那些需要随文档一起返回但不需要被搜索的数据：

```
PUT app_logs
{
  "mappings": {
    "properties": {
      "message": { "type": "text" },
      "level": { "type": "keyword" },
      "raw_request": {
        "type": "object",
        "enabled": false
      }
    }
  }
}
```

在这个例子中，`raw_request` 中存储了完整的 HTTP 请求体，便于调试时查看原始数据，但不会对其内容建立索引。

### 禁用整个映射的索引

还可以在顶层映射上使用 `enabled: false`，使整个索引变成纯存储模式：

```
PUT archive
{
  "mappings": {
    "enabled": false
  }
}
```

这会让索引接受任意 JSON 数据而不解析字段，适合纯归档用途。

## 注意事项

| 行为                   | `enabled: true`（默认） | `enabled: false`       |
| ---------------------- | ----------------------- | ---------------------- |
| 搜索（match/term/...） | ✅ 支持                 | ❌ 不支持               |
| 排序和聚合             | ✅ 支持                 | ❌ 不支持               |
| 从 `_source` 获取      | ✅ 支持                 | ✅ 支持                 |
| 动态映射               | ✅ 自动检测字段类型     | ❌ 不检测               |

> **注意**：`enabled` 参数在索引创建后无法修改。如需更改，必须重建索引。

