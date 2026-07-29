---
title: "空值参数（Null Value）"
date: 0001-01-01
summary: "Null Value 参数 #  null_value 参数指定一个替代值，用于在字段值为 null 或缺失时代替索引。这使得 null 值可以被搜索和聚合。
相关指南（先读这些） #    映射基础  结构化搜索  基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;status&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;null_value&#34;: &#34;UNKNOWN&#34; } } } } 写入和查询：
PUT my-index/_doc/1 { &#34;status&#34;: null } PUT my-index/_doc/2 { &#34;status&#34;: &#34;active&#34; } # 搜索 null 值的文档 GET my-index/_search { &#34;query&#34;: { &#34;term&#34;: { &#34;status&#34;: &#34;UNKNOWN&#34; } } } 文档 1 会被上面的查询找到，因为 null 被替换为 &quot;UNKNOWN&quot; 进行索引。"
---


# Null Value 参数

`null_value` 参数指定一个替代值，用于在字段值为 `null` 或缺失时代替索引。这使得 `null` 值可以被搜索和聚合。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "status": {
        "type": "keyword",
        "null_value": "UNKNOWN"
      }
    }
  }
}
```

写入和查询：

```json
PUT my-index/_doc/1
{ "status": null }

PUT my-index/_doc/2
{ "status": "active" }

# 搜索 null 值的文档
GET my-index/_search
{
  "query": {
    "term": { "status": "UNKNOWN" }
  }
}
```

文档 1 会被上面的查询找到，因为 `null` 被替换为 `"UNKNOWN"` 进行索引。

## 重要行为

| 行为 | 说明 |
|------|------|
| _source | `_source` 中仍然显示原始的 `null` 值，**不会**被替换 |
| 索引 | 替换值会被写入倒排索引和 doc_values |
| 类型匹配 | 替换值的类型**必须**与字段类型一致 |
| 默认值 | 默认为 `null`（即不替换，null 值的字段被视为缺失） |

## 按字段类型的示例

```json
"properties": {
  "count": {
    "type": "integer",
    "null_value": 0
  },
  "is_active": {
    "type": "boolean",
    "null_value": false
  },
  "location": {
    "type": "geo_point",
    "null_value": {
      "lat": 0,
      "lon": 0
    }
  }
}
```

## 与 exists 查询的关系

如果**没有**设置 `null_value`，null 或缺失的字段可以通过 `exists` 查询的否定来查找：

```json
{
  "query": {
    "bool": {
      "must_not": {
        "exists": { "field": "status" }
      }
    }
  }
}
```

如果**设置了** `null_value`，那么 null 值的文档也会被 `exists` 查询匹配到（因为有一个替代值被索引了）。

## 注意事项

- `null_value` 仅影响索引和搜索，**不影响** `_source` 中的原始值
- 空数组 `[]` 和显式 `null` 的处理方式相同
- 对 `text` 字段设置 `null_value` 没有意义（因为 text 字段不支持精确匹配）
- 该参数可以通过 `PUT mapping` API 更新

