---
title: "Lowercase 规范化器"
date: 0001-01-01
summary: "Lowercase 规范化器 #  lowercase 是 Easysearch 内置的规范化器，将 keyword 字段的整个值转换为小写形式。无需在 settings 中定义即可直接使用。
使用方式 #  在映射中直接引用：
PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;status&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;normalizer&#34;: &#34;lowercase&#34; } } } } 效果演示 #  索引文档：
PUT my-index/_doc/1 { &#34;status&#34;: &#34;OK&#34; } PUT my-index/_doc/2 { &#34;status&#34;: &#34;Ok&#34; } PUT my-index/_doc/3 { &#34;status&#34;: &#34;ok&#34; } 查询时，无论使用哪种大小写形式都能匹配所有文档：
GET my-index/_search { &#34;query&#34;: { &#34;term&#34;: { &#34;status&#34;: &#34;OK&#34; } } } 三个文档都会被返回，因为 &quot;OK&quot;、&quot;Ok&quot;、&quot;ok&quot; 在索引时都被归一化为 &quot;ok&quot;。"
---


# Lowercase 规范化器

`lowercase` 是 Easysearch 内置的规范化器，将 keyword 字段的整个值转换为小写形式。无需在 settings 中定义即可直接使用。

## 使用方式

在映射中直接引用：

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "status": {
        "type": "keyword",
        "normalizer": "lowercase"
      }
    }
  }
}
```

## 效果演示

索引文档：

```json
PUT my-index/_doc/1
{
  "status": "OK"
}

PUT my-index/_doc/2
{
  "status": "Ok"
}

PUT my-index/_doc/3
{
  "status": "ok"
}
```

查询时，无论使用哪种大小写形式都能匹配所有文档：

```json
GET my-index/_search
{
  "query": {
    "term": {
      "status": "OK"
    }
  }
}
```

三个文档都会被返回，因为 `"OK"`、`"Ok"`、`"ok"` 在索引时都被归一化为 `"ok"`。

## 对聚合的影响

```json
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "status_counts": {
      "terms": {
        "field": "status"
      }
    }
  }
}
```

聚合结果中只会出现 `"ok"` 一个桶（而非三个不同的大小写变体），计数为 3。

## 等价的自定义配置

`lowercase` 内置规范化器等价于以下自定义配置：

```json
PUT my-index
{
  "settings": {
    "analysis": {
      "normalizer": {
        "my_lowercase": {
          "type": "custom",
          "filter": ["lowercase"]
        }
      }
    }
  }
}
```

如果只需要小写转换，推荐直接使用内置的 `lowercase`，更简洁。

## 适用场景

| 场景 | 示例 |
|------|------|
| 状态码匹配 | `"OK"` / `"ok"` / `"Ok"` 视为相同 |
| 标签归一化 | `"JavaScript"` / `"javascript"` 聚合在一起 |
| 枚举值查询 | `"ACTIVE"` / `"active"` 无差别匹配 |
| 用户名不区分大小写 | `"Admin"` / `"admin"` 指向同一用户 |

## 限制

- 仅做小写转换，不处理变音符号（`"Naïve"` → `"naïve"` 而非 `"naive"`）
- 不做 Unicode 归一化或字符映射
- 如需更复杂的归一化逻辑，请使用[自定义规范化器]({{< relref "./custom" >}})

