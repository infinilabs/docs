---
title: "属性参数（Properties）"
date: 0001-01-01
summary: "Properties 参数 #  properties 参数用于定义对象（object）和嵌套（nested）类型字段的子字段映射。它是构建层级文档结构的核心参数。
相关指南 #    映射基础  dynamic 参数  使用位置 #  properties 出现在三个层级：
   层级 说明     mappings.properties 顶层字段定义   mappings.properties.&lt;object&gt;.properties 对象字段的子字段   mappings.properties.&lt;nested&gt;.properties 嵌套字段的子字段    示例 #  基本层级结构 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;name&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;address&#34;: { &#34;type&#34;: &#34;object&#34;, &#34;properties&#34;: { &#34;city&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;zip&#34;: { &#34;type&#34;: &#34;keyword&#34; } } } } } } 嵌套类型 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;comments&#34;: { &#34;type&#34;: &#34;nested&#34;, &#34;properties&#34;: { &#34;author&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;content&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;date&#34;: { &#34;type&#34;: &#34;date&#34; } } } } } } 多层嵌套 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;department&#34;: { &#34;properties&#34;: { &#34;name&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;manager&#34;: { &#34;properties&#34;: { &#34;name&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;email&#34;: { &#34;type&#34;: &#34;keyword&#34; } } } } } } } } 对应的文档字段以 ."
---


# Properties 参数

`properties` 参数用于定义对象（`object`）和嵌套（`nested`）类型字段的**子字段**映射。它是构建层级文档结构的核心参数。

## 相关指南

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [dynamic 参数]({{< relref "/docs/features/mapping-and-analysis/mapping-parameters/dynamic.md" >}})

## 使用位置

`properties` 出现在三个层级：

| 层级 | 说明 |
|------|------|
| `mappings.properties` | 顶层字段定义 |
| `mappings.properties.<object>.properties` | 对象字段的子字段 |
| `mappings.properties.<nested>.properties` | 嵌套字段的子字段 |

## 示例

### 基本层级结构

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "address": {
        "type": "object",
        "properties": {
          "city": { "type": "keyword" },
          "zip": { "type": "keyword" }
        }
      }
    }
  }
}
```

### 嵌套类型

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested",
        "properties": {
          "author": { "type": "keyword" },
          "content": { "type": "text" },
          "date": { "type": "date" }
        }
      }
    }
  }
}
```

### 多层嵌套

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "department": {
        "properties": {
          "name": { "type": "keyword" },
          "manager": {
            "properties": {
              "name": { "type": "text" },
              "email": { "type": "keyword" }
            }
          }
        }
      }
    }
  }
}
```

对应的文档字段以 `.` 分隔：`department.manager.email`。

## object 与 nested 的区别

| 特性 | object（默认） | nested |
|------|---------------|--------|
| 内部对象独立性 | 数组中的对象会被扁平化，失去关联关系 | 每个对象独立存储，保持内部字段关联 |
| 查询方式 | 普通查询 | 需要使用 `nested` 查询 |
| 性能 | 更快 | 较慢（每个嵌套对象作为独立 Lucene 文档） |
| 适用场景 | 非数组对象或不需要精确关联的数组 | 需要精确匹配数组中特定对象内字段关联 |

## 注意事项

- `properties` 不需要指定 `type`，它本身就是对象结构的定义方式
- 省略 `type` 时，字段自动被视为 `object` 类型
- 已有字段可以新增子字段（通过 Update Mapping API），但已有子字段的类型不可更改
- 嵌套对象过多会影响索引和查询性能，建议通过 `index.mapping.nested_objects.limit` 控制数量

