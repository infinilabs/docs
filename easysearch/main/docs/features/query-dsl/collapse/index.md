---
title: "结果折叠"
date: 0001-01-01
description: "Field Collapsing：按字段值对搜索结果去重折叠，每组只展示 Top-N。"
summary: "结果折叠 #  结果折叠（Field Collapsing）允许你按某个字段的值对搜索结果进行分组去重，每组只返回最相关的一条（或通过 inner_hits 展开多条）。常用于：
 商品搜索：同一品牌/店铺只展示最相关的一个商品 新闻搜索：同一来源的新闻只展示一条 日志分析：按主机名折叠，每台机器只显示最新的一条  基础用法 #  GET products/_search { &#34;query&#34;: { &#34;match&#34;: { &#34;title&#34;: &#34;手机&#34; } }, &#34;collapse&#34;: { &#34;field&#34;: &#34;brand.keyword&#34; }, &#34;sort&#34;: [{ &#34;_score&#34;: &#34;desc&#34; }] } 每个 brand.keyword 值只返回分数最高的一条文档。
 字段要求：collapse 字段必须是 keyword 或数值类型，且必须启用 doc_values。
 参数说明 #     参数 类型 默认值 说明     field String 必填 用于折叠的字段名（keyword 或数值类型，需有 doc_values）   inner_hits Object 或 Array 空 展开折叠组，返回组内的多条文档   max_concurrent_group_searches Integer 0（不限制） 内部 inner_hits 请求的最大并发数    使用 inner_hits 展开 #  默认每组只返回一条文档。通过 inner_hits 可以展开每组，返回组内的 Top-N 文档："
---


# 结果折叠

结果折叠（Field Collapsing）允许你按某个字段的值对搜索结果进行**分组去重**，每组只返回最相关的一条（或通过 `inner_hits` 展开多条）。常用于：

- 商品搜索：同一品牌/店铺只展示最相关的一个商品
- 新闻搜索：同一来源的新闻只展示一条
- 日志分析：按主机名折叠，每台机器只显示最新的一条

## 基础用法

```json
GET products/_search
{
  "query": {
    "match": { "title": "手机" }
  },
  "collapse": {
    "field": "brand.keyword"
  },
  "sort": [{ "_score": "desc" }]
}
```

每个 `brand.keyword` 值只返回分数最高的一条文档。

> **字段要求**：`collapse` 字段必须是 `keyword` 或数值类型，且必须启用 `doc_values`。

## 参数说明

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `field` | String | 必填 | 用于折叠的字段名（keyword 或数值类型，需有 doc_values） |
| `inner_hits` | Object 或 Array | 空 | 展开折叠组，返回组内的多条文档 |
| `max_concurrent_group_searches` | Integer | 0（不限制） | 内部 inner_hits 请求的最大并发数 |

## 使用 inner_hits 展开

默认每组只返回一条文档。通过 `inner_hits` 可以展开每组，返回组内的 Top-N 文档：

```json
GET products/_search
{
  "query": {
    "match": { "title": "手机" }
  },
  "collapse": {
    "field": "brand.keyword",
    "inner_hits": {
      "name": "brand_top3",
      "size": 3,
      "sort": [{ "price": "asc" }]
    }
  },
  "sort": [{ "_score": "desc" }]
}
```

### inner_hits 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `name` | String | - | inner_hits 的名称标识 |
| `size` | Integer | 3 | 每组返回的文档数 |
| `from` | Integer | 0 | 组内偏移量 |
| `sort` | Array | - | 组内排序（可以与外层排序不同） |
| `_source` | Boolean/Object | - | 控制返回的 _source 字段 |
| `highlight` | Object | - | 组内文档的高亮配置 |
| `explain` | Boolean | false | 是否返回评分解释 |
| `stored_fields` | Array | - | 返回的存储字段列表 |
| `docvalue_fields` | Array | - | 返回的 doc_value 字段列表 |
| `script_fields` | Object | - | 脚本字段 |
| `track_scores` | Boolean | false | 是否计算分数 |
| `version` | Boolean | false | 是否返回文档版本 |
| `seq_no_primary_term` | Boolean | false | 是否返回 seq_no 和 primary_term |

## 多个 inner_hits

可以为同一个折叠字段配置多组 `inner_hits`，各自独立排序：

```json
GET products/_search
{
  "query": {
    "match": { "title": "手机" }
  },
  "collapse": {
    "field": "brand.keyword",
    "inner_hits": [
      {
        "name": "most_relevant",
        "size": 3,
        "sort": [{ "_score": "desc" }]
      },
      {
        "name": "cheapest",
        "size": 3,
        "sort": [{ "price": "asc" }]
      }
    ],
    "max_concurrent_group_searches": 4
  }
}
```

## 二级折叠

`inner_hits` 内部还可以再嵌套一层 `collapse`，实现二级折叠：

```json
GET products/_search
{
  "query": {
    "match": { "title": "手机" }
  },
  "collapse": {
    "field": "brand.keyword",
    "inner_hits": {
      "name": "by_model",
      "size": 3,
      "collapse": {
        "field": "model.keyword"
      }
    }
  }
}
```

> **限制**：二级折叠只支持 `field` 参数，不支持进一步嵌套 `inner_hits`。

## 注意事项

- 折叠在 `_score` 排序和自定义排序下都有效
- 折叠不影响聚合（aggregations），聚合仍然基于全部匹配文档计算
- 响应中的 `hits.total` 是折叠前的匹配文档总数，而非折叠后的组数
- 折叠后不支持 scroll 和 rescore
- 如果折叠字段的值为 `null` 或缺失，该文档会被分到一个"空值组"

