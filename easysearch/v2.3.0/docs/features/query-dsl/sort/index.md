---
title: "排序"
date: 0001-01-01
description: "字段排序、多级排序、地理距离排序、脚本排序、missing 值处理的 API 与参数说明。"
summary: "排序 #  默认情况下，全文查询按相关性 _score 排序。你也可以按任意字段值升序/降序排序，或使用地理距离、脚本等高级排序方式。
相关指南 #    分页与排序   基础字段排序 #  通过 sort 参数指定排序字段和顺序：
GET shakespeare/_search { &#34;query&#34;: { &#34;term&#34;: { &#34;play_name&#34;: { &#34;value&#34;: &#34;Henry IV&#34; } } }, &#34;sort&#34;: [ { &#34;line_id&#34;: { &#34;order&#34;: &#34;desc&#34; } } ] } 排序参数 #     参数 说明 可选值     order 排序方向 asc（升序）、desc（降序）   mode 多值字段的聚合模式 min、max、avg、sum、median   missing 缺失值的处理方式 _first（排最前）、_last（排最后）、自定义值   unmapped_type 未映射字段的假设类型 字段类型名（如 long、date）     多级排序 #  sort 是数组，可以配置多级排序。当主排序键相同时，使用次排序键："
---


# 排序

默认情况下，全文查询按相关性 `_score` 排序。你也可以按任意字段值升序/降序排序，或使用地理距离、脚本等高级排序方式。

## 相关指南

- [分页与排序]({{< relref "/docs/features/fulltext-search/pagination-and-sorting.md" >}})

---

## 基础字段排序

通过 `sort` 参数指定排序字段和顺序：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "play_name": { "value": "Henry IV" }
    }
  },
  "sort": [
    { "line_id": { "order": "desc" } }
  ]
}
```

### 排序参数

| 参数 | 说明 | 可选值 |
|------|------|--------|
| `order` | 排序方向 | `asc`（升序）、`desc`（降序） |
| `mode` | 多值字段的聚合模式 | `min`、`max`、`avg`、`sum`、`median` |
| `missing` | 缺失值的处理方式 | `_first`（排最前）、`_last`（排最后）、自定义值 |
| `unmapped_type` | 未映射字段的假设类型 | 字段类型名（如 `long`、`date`） |

---

## 多级排序

`sort` 是数组，可以配置多级排序。当主排序键相同时，使用次排序键：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "play_name": { "value": "Henry IV" }
    }
  },
  "sort": [
    { "line_id": { "order": "desc" } },
    { "speech_number": { "order": "desc" } }
  ]
}
```

---

## 多值字段排序（mode）

对于数组类型的数值字段，通过 `mode` 选择聚合值用于排序：

```json
GET products/_search
{
  "sort": [
    {
      "price": {
        "order": "asc",
        "mode": "avg"
      }
    }
  ]
}
```

---

## 缺失值处理（missing）

控制当文档缺少排序字段时的排列位置：

```json
GET products/_search
{
  "sort": [
    {
      "price": {
        "order": "asc",
        "missing": "_last"
      }
    }
  ]
}
```

---

## 按 keyword 字段排序

被分析过的 `text` 字段不能直接排序，因为倒排索引只包含拆分后的 term。常见做法是使用 `keyword` 子字段：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "play_name": { "value": "Henry IV" }
    }
  },
  "sort": [
    { "play_name.keyword": { "order": "desc" } }
  ]
}
```

---

## 地理距离排序（_geo_distance）

按地理位置距离排序，适合"附近的店铺"等场景：

```json
GET restaurants/_search
{
  "sort": [
    {
      "_geo_distance": {
        "location": { "lat": 40.715, "lon": -73.998 },
        "order": "asc",
        "unit": "km",
        "distance_type": "arc"
      }
    }
  ]
}
```

### _geo_distance 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `location`（字段名） | 参考坐标点 | 必填 |
| `order` | 排序方向 | `asc` |
| `unit` | 距离单位（`km`、`m`、`mi`、`yd` 等） | `m` |
| `distance_type` | 距离计算方式 | `arc`（精确）或 `plane`（快速近似） |
| `mode` | 多点字段的取值模式 | `min` |

---

## 脚本排序（_script）

使用 Painless 脚本自定义排序逻辑：

```json
GET products/_search
{
  "sort": [
    {
      "_script": {
        "type": "number",
        "script": {
          "source": "doc['price'].value * doc['discount'].value"
        },
        "order": "asc"
      }
    }
  ]
}
```

### _script 参数

| 参数 | 说明 |
|------|------|
| `type` | 排序值类型（`number` 或 `string`） |
| `script.source` | Painless 脚本 |
| `script.params` | 脚本参数 |
| `order` | 排序方向 |

---

## 嵌套字段排序（nested）

对 nested 对象内的字段排序：

```json
GET products/_search
{
  "sort": [
    {
      "reviews.rating": {
        "order": "desc",
        "nested": {
          "path": "reviews",
          "filter": {
            "term": { "reviews.verified": true }
          }
        }
      }
    }
  ]
}
```

`nested.filter` 可选，用于只对满足条件的嵌套文档参与排序计算。

---

## 按 _score 排序

显式按相关性分数排序（默认行为），通常配合其他排序字段使用：

```json
GET shakespeare/_search
{
  "query": {
    "match": { "text_entry": "to be" }
  },
  "sort": [
    { "_score": { "order": "desc" } },
    { "line_id": { "order": "asc" } }
  ]
}
```

> **提示**：使用自定义 `sort` 后，默认不再返回 `_score`。如果需要同时获取相关性分数，设置 `"track_scores": true`。

