---
title: "位置增量保留（Preserve Position Increments）"
date: 0001-01-01
summary: "位置增量保留（Preserve Position Increments） #  确定是否在自动完成建议中保留位置增量。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;suggest&#34;: { &#34;type&#34;: &#34;completion&#34;, &#34;preserve_position_increments&#34;: true } } } } 适用的字段类型 #   completion  默认值 #   true  说明 #  当启用时，停用词等被移除的术语所创建的位置间隙被保留。这会影响自动完成建议的精确性和性能。
相关参数 #    分隔符保留（Preserve Separators）  "
---


# 位置增量保留（Preserve Position Increments）

确定是否在自动完成建议中保留位置增量。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "suggest": {
        "type": "completion",
        "preserve_position_increments": true
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

当启用时，停用词等被移除的术语所创建的位置间隙被保留。这会影响自动完成建议的精确性和性能。

## 相关参数

- [分隔符保留（Preserve Separators）]({{< relref "preserve_separators.md" >}})

