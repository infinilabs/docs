---
title: "对象字段类型（Object）"
date: 0001-01-01
summary: "Object 对象字段类型 #  本页列出对象与关系型字段类型的索引。
如何在建模时选择 object / nested / join，请优先阅读数据建模章节中的 Guide 文档。
相关指南（先读这些） #    映射基础  映射模式  Nested 建模  Parent-Child 建模  字段类型总览 #  对象字段类型包含对象或关系类型的值。下表列出了 Easysearch 支持的所有对象字段类型。
   字段数据类型 描述     object JSON 对象。   nested 用于数组中的对象需要作为有关系的文档单独索引时。   flattened 作为字符串处理的 JSON 对象。   join 在同一索引中的文档之间建立父/子关系。    "
---


# Object 对象字段类型

本页列出对象与关系型字段类型的索引。  
如何在建模时选择 object / nested / join，请优先阅读数据建模章节中的 Guide 文档。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})
- [Nested 建模]({{< relref "/docs/best-practices/data-modeling/nested.md" >}})
- [Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})

## 字段类型总览

对象字段类型包含对象或关系类型的值。下表列出了 Easysearch 支持的所有对象字段类型。

| 字段数据类型  | 描述                                             |
| ------------- | ------------------------------------------------ |
| `object`      | JSON 对象。                                      |
| `nested`      | 用于数组中的对象需要作为有关系的文档单独索引时。 |
| `flattened` | 作为字符串处理的 JSON 对象。                     |
| `join`        | 在同一索引中的文档之间建立父/子关系。            |
