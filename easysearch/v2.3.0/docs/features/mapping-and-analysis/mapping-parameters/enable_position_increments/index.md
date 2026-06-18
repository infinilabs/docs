---
title: "位置增量启用（Enable Position Increments）"
date: 0001-01-01
summary: "位置增量启用（Enable Position Increments） #  确定是否在令牌计数中包含位置增量。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;token_count&#34;, &#34;analyzer&#34;: &#34;standard&#34;, &#34;enable_position_increments&#34;: true } } } } 适用的字段类型 #   token_count（mapper-extras 模块）  默认值 #   true  说明 #  当启用时，停用词等被移除的令牌所创建的位置间隙被计入令牌计数。当禁用时，只计算实际产生的令牌。
相关参数 #    分析器参数（Analyzer）  "
---


# 位置增量启用（Enable Position Increments）

确定是否在令牌计数中包含位置增量。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "token_count",
        "analyzer": "standard",
        "enable_position_increments": true
      }
    }
  }
}
```

## 适用的字段类型

- `token_count`（mapper-extras 模块）

## 默认值

- `true`

## 说明

当启用时，停用词等被移除的令牌所创建的位置间隙被计入令牌计数。当禁用时，只计算实际产生的令牌。

## 相关参数

- [分析器参数（Analyzer）]({{< relref "analyzer.md" >}})

