---
title: "评分影响方向（Positive Score Impact）"
date: 0001-01-01
summary: "评分影响方向（Positive Score Impact） #  指示排名特性对得分的影响方向。
基本用法 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;boost&#34;: { &#34;type&#34;: &#34;rank_feature&#34;, &#34;positive_score_impact&#34;: true } } } } 适用的字段类型 #   rank_feature（mapper-extras 模块）  默认值 #   true  说明 #   true: 特性值越高，文档得分越高（正相关） false: 特性值越高，文档得分越低（负相关）  此参数告知搜索引擎如何使用排名特性来影响文档评分。
相关参数 #    索引参数（Index）  "
---


# 评分影响方向（Positive Score Impact）

指示排名特性对得分的影响方向。

## 基本用法

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "boost": {
        "type": "rank_feature",
        "positive_score_impact": true
      }
    }
  }
}
```

## 适用的字段类型

- `rank_feature`（mapper-extras 模块）

## 默认值

- `true`

## 说明

- **true**: 特性值越高，文档得分越高（正相关）
- **false**: 特性值越高，文档得分越低（负相关）

此参数告知搜索引擎如何使用排名特性来影响文档评分。

## 相关参数

- [索引参数（Index）]({{< relref "index.md" >}})

