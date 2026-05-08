---
title: "Nested 建模"
date: 0001-01-01
description: "嵌套对象的建模、查询与聚合。"
tags: ["最佳实践", "数据建模"]
summary: "Nested 建模 #  Nested 解决的是这样一种典型需求：数组里是一组对象，而不是一堆无关字段的拼接，查询时既要对数组元素内部做精确匹配，又不想被&quot;笛卡尔积假匹配&quot;坑到。
什么时候需要 nested？ #  先看一个常见例子：订单里有多条明细 items：
{ &#34;order_id&#34;: &#34;O-1&#34;, &#34;items&#34;: [ { &#34;sku&#34;: &#34;A&#34;, &#34;price&#34;: 100, &#34;qty&#34;: 1 }, { &#34;sku&#34;: &#34;B&#34;, &#34;price&#34;: 200, &#34;qty&#34;: 2 } ] } 如果你把 items.sku、items.price、items.qty 都当成普通多值字段：
 items.sku = [&ldquo;A&rdquo;, &ldquo;B&rdquo;] items.price = [100, 200] items.qty = [1, 2]  此时一个查询：
 items.sku = &quot;A&quot; 且 items.price = 200  在&quot;扁平多值字段&quot;模型下是会命中的（因为它只在每个字段内部看是否包含该值），但现实里并不存在 sku=A 且 price=200 这一条明细。这就是经典的&quot;笛卡尔积假匹配&quot;。
要避免这种问题，就需要 nested。
如何定义 nested 字段 #  在 Mapping 中，把数组元素声明为 nested 类型（示意）："
---


# Nested 建模

Nested 解决的是这样一种典型需求：**数组里是一组对象，而不是一堆无关字段的拼接**，查询时既要对数组元素内部做精确匹配，又不想被"笛卡尔积假匹配"坑到。

## 什么时候需要 nested？

先看一个常见例子：订单里有多条明细 `items`：

```json
{
  "order_id": "O-1",
  "items": [
    { "sku": "A", "price": 100, "qty": 1 },
    { "sku": "B", "price": 200, "qty": 2 }
  ]
}
```

如果你把 `items.sku`、`items.price`、`items.qty` 都当成普通多值字段：

- `items.sku` = ["A", "B"]
- `items.price` = [100, 200]
- `items.qty` = [1, 2]

此时一个查询：

- `items.sku = "A"` 且 `items.price = 200`

在"扁平多值字段"模型下是**会命中**的（因为它只在每个字段内部看是否包含该值），但现实里并不存在 `sku=A 且 price=200` 这一条明细。这就是经典的"笛卡尔积假匹配"。

要避免这种问题，就需要 nested。

## 如何定义 nested 字段

在 Mapping 中，把数组元素声明为 `nested` 类型（示意）：

```json
{
  "mappings": {
    "properties": {
      "order_id": { "type": "keyword" },
      "items": {
        "type": "nested",
        "properties": {
          "sku":   { "type": "keyword" },
          "price": { "type": "double"  },
          "qty":   { "type": "integer" }
        }
      }
    }
  }
}
```

含义：

- `items` 里的每个元素在底层会作为一个"子文档"存储
- nested 查询可以在"同一个子文档"内部组合多个条件，从而避免混淆

## nested 查询的基本用法

仍以"查找包含 `sku=A 且 price=200` 的订单"为例：

```json
{
  "query": {
    "nested": {
      "path": "items",
      "query": {
        "bool": {
          "must": [
            { "term":  { "items.sku":   "A"   } },
            { "range": { "items.price": { "gte": 200 } } }
          ]
        }
      }
    }
  }
}
```

关键点：

- `path`：指定 nested 字段路径（这里是 `items`）
- `query`：在该 path 内部再写一套 Query DSL（通常用 bool）

查询逻辑：

- 先在所有 `items` 子文档中查找满足条件的元素
- 再把命中的子文档"折叠"回其所属父文档，返回对应的订单

## nested 聚合

如果你想对 nested 字段里的数据做聚合（例如统计所有明细中的 SKU 分布），需要显式进入 nested 上下文（示意）：

```json
{
  "aggs": {
    "items": {
      "nested": {
        "path": "items"
      },
      "aggs": {
        "by_sku": {
          "terms": {
            "field": "items.sku"
          }
        }
      }
    }
  }
}
```

如果还要回到父文档维度再做聚合，可配合 `reverse_nested` 使用。

## nested 的代价与取舍

优点：

- 消除笛卡尔积假匹配，语义清晰
- 支持在数组元素内做复杂组合条件与聚合

成本：

- nested 元素在底层是"多个子文档"，文档数量会放大
- 查询、聚合在 nested 上下文中会有额外开销
- 设计不当时，可能让索引体积与查询复杂度都变大

经验建议：

- nested 非常适合"真正的一组对象数组"，且数组长度适中（例如订单明细、标签属性等）
- 如果数组非常长、结构复杂，且查询场景简单，考虑用反范式/预计算字段替代

## 小结

- Nested 解决数组里是一组对象时的"笛卡尔积假匹配"问题
- 在 Mapping 中把数组元素声明为 `nested` 类型
- nested 查询可以在"同一个子文档"内部组合多个条件
- nested 聚合需要显式进入 nested 上下文
- nested 的代价是文档数量放大和查询开销增加

下一步可以继续阅读：

- [Parent-Child 建模](./parent-child.md)
- [反范式与权衡](./denormalization.md)
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

## 参考手册（API 与参数）

- [关联查询（功能手册）]({{< relref "/docs/features/query-dsl/joining/_index.md" >}})
- [Nested 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/object-field-type/nested.md" >}})

