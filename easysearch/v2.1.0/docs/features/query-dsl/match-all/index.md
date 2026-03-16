---
title: "Match All 查询"
date: 0001-01-01
summary: "Match All 查询 #  match_all 查询返回所有文档。如果需要返回整个文档集，这个查询在测试大量文档集时很有用。
相关指南（先读这些） #    查询 DSL 基础  结构化搜索  GET _search { &#34;query&#34;: { &#34;match_all&#34;: {} } } match_all 查询有一个 match_none 的对应查询，这个对应查询很少有用：
GET _search { &#34;query&#34;: { &#34;match_none&#34;: {} } } 参数说明 #  全匹配和全不匹配查询都接受以下参数。所有参数都是可选的。
   参数 数据类型 描述     boost Float 一个浮点数值，用于指定该字段对相关性分数的权重。大于 1.0 的值会增加字段的权重。介于 0.0 和 1.0 之间的值会降低字段的权重。默认值为 1.0。   _name String 用于查询标签的查询名称。可选。    常见使用场景 #  配合 filter 使用 #  match_all 经常与过滤器结合使用，在不需要相关性评分的场景下获取文档子集："
---


# Match All 查询

`match_all` 查询返回所有文档。如果需要返回整个文档集，这个查询在测试大量文档集时很有用。

## 相关指南（先读这些）

- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

```
GET _search
{
  "query": {
    "match_all": {}
  }
}

```

`match_all` 查询有一个 `match_none` 的对应查询，这个对应查询很少有用：

```
GET _search
{
  "query": {
    "match_none": {}
  }
}

```

## 参数说明

全匹配和全不匹配查询都接受以下参数。所有参数都是可选的。
|参数|数据类型|描述|
| --- | --- | --- |
|`boost`|Float|一个浮点数值，用于指定该字段对相关性分数的权重。大于 1.0 的值会增加字段的权重。介于 0.0 和 1.0 之间的值会降低字段的权重。默认值为 1.0。|
|`_name`|String|用于查询标签的查询名称。可选。|

## 常见使用场景

### 配合 filter 使用

`match_all` 经常与过滤器结合使用，在不需要相关性评分的场景下获取文档子集：

```
GET products/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "price": { "gte": 100 }
        }
      }
    }
  }
}
```

### 在聚合中使用

当只需要聚合结果而不关心搜索命中时，`match_all` 是默认的隐含查询。以下两种写法等价：

```
GET products/_search
{
  "size": 0,
  "aggs": {
    "avg_price": {
      "avg": { "field": "price" }
    }
  }
}
```

### 调整 boost 控制评分

通过调整 `boost` 值可以控制 `match_all` 分配给所有文档的评分（默认为 1.0）：

```
GET _search
{
  "query": {
    "match_all": { "boost": 1.5 }
  }
}
```

> **提示**：`match_all` 在没有指定查询条件时是 Easysearch 的默认行为。直接使用 `GET index/_search` 等价于使用 `match_all`。

