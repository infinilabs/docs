---
title: "分隔符保留（Preserve Separators）"
date: 0001-01-01
summary: "分隔符保留（Preserve Separators） #  确定在自动完成建议中是否保留分隔符。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;suggest&#34;: { &#34;type&#34;: &#34;completion&#34;, &#34;preserve_separators&#34;: true } } } } 适用的字段类型 #   completion  默认值 #   true  说明 #  当启用时，分隔符（如空格、连字符和斜杠）被保留为单独的项。这有助于匹配在分隔符边界处开始的建议。
相关参数 #    位置增量保留（Preserve Position Increments）  "
---


# 分隔符保留（Preserve Separators）

确定在自动完成建议中是否保留分隔符。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "suggest": {
        "type": "completion",
        "preserve_separators": true
      }
    }
  }
}
```

## 适用的字段类型

- `completion`

## 默认值

- `true`

## 说明

当启用时，分隔符（如空格、连字符和斜杠）被保留为单独的项。这有助于匹配在分隔符边界处开始的建议。

## 相关参数

- [位置增量保留（Preserve Position Increments）]({{< relref "preserve_position_increments.md" >}})

