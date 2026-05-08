---
title: "完成上下文（Contexts）"
date: 0001-01-01
summary: "完成上下文（Contexts） #  定义用于过滤自动完成建议的上下文。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;suggest&#34;: { &#34;type&#34;: &#34;completion&#34;, &#34;contexts&#34;: [ { &#34;name&#34;: &#34;category&#34;, &#34;type&#34;: &#34;category&#34; }, { &#34;name&#34;: &#34;location&#34;, &#34;type&#34;: &#34;geo&#34;, &#34;precision&#34;: &#34;5km&#34; } ] } } } } 适用的字段类型 #   completion  上下文类型 #   category: 按类别过滤建议 geo: 按地理位置过滤建议  说明 #  上下文允许根据多个维度（如类别或地理位置）对完成建议进行分类和过滤。这在需要基于上下文的自动完成的应用中很有用。
相关参数 #    分隔符保留（Preserve Separators）  位置增量保留（Preserve Position Increments）  "
---


# 完成上下文（Contexts）

定义用于过滤自动完成建议的上下文。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "suggest": {
        "type": "completion",
        "contexts": [
          {
            "name": "category",
            "type": "category"
          },
          {
            "name": "location",
            "type": "geo",
            "precision": "5km"
          }
        ]
      }
    }
  }
}
```

## 适用的字段类型

- `completion`

## 上下文类型

- **category**: 按类别过滤建议
- **geo**: 按地理位置过滤建议

## 说明

上下文允许根据多个维度（如类别或地理位置）对完成建议进行分类和过滤。这在需要基于上下文的自动完成的应用中很有用。

## 相关参数

- [分隔符保留（Preserve Separators）]({{< relref "preserve_separators.md" >}})
- [位置增量保留（Preserve Position Increments）]({{< relref "preserve_position_increments.md" >}})

