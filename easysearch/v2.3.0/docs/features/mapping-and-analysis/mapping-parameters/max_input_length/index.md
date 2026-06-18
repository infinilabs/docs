---
title: "最大输入长度（Max Input Length）"
date: 0001-01-01
summary: "最大输入长度（Max Input Length） #  限制自动完成建议的最大输入长度。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;suggest&#34;: { &#34;type&#34;: &#34;completion&#34;, &#34;max_input_length&#34;: 50 } } } } 适用的字段类型 #   completion  默认值 #   50  说明 #  当输入超过指定的长度时，将被截断。这用于限制自动完成索引的大小和内存使用。
相关参数 #    完成上下文（Contexts）  "
---


# 最大输入长度（Max Input Length）

限制自动完成建议的最大输入长度。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "suggest": {
        "type": "completion",
        "max_input_length": 50
      }
    }
  }
}
```

## 适用的字段类型

- `completion`

## 默认值

- `50`

## 说明

当输入超过指定的长度时，将被截断。这用于限制自动完成索引的大小和内存使用。

## 相关参数

- [完成上下文（Contexts）]({{< relref "contexts.md" >}})

