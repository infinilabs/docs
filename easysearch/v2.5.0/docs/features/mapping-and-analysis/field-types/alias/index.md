---
title: "别名字段类型（Alias）"
date: 0001-01-01
summary: "Alias 别名字段类型 #  别名字段类型为现有字段创建另一个名称。您可以在搜索和字段功能的 API 操作中使用别名字段，但存在一些例外情况。要设置别名，必须在 path 参数中指定原始字段名称。
参考代码 #  PUT movies { &#34;mappings&#34; : { &#34;properties&#34; : { &#34;year&#34; : { &#34;type&#34; : &#34;date&#34; }, &#34;release_date&#34; : { &#34;type&#34; : &#34;alias&#34;, &#34;path&#34; : &#34;year&#34; } } } } 参数说明 #   path：指向原始字段的完整路径，包括所有父对象。例如，parent.child.field_name。此参数为必填项。  别名（Alias）字段 #  别名（Alias）字段必须遵循以下规则：
 一个别名字段只能引用一个原始字段。 在嵌套对象中，别名必须与原始字段位于相同的嵌套层级。   要更改别名引用的字段，需要更新映射配置。但请注意，之前存储的 Percolator 查询中的别名仍会继续引用原始字段，不会自动更新为新的字段引用。
 原始字段 #  别名的原始字段必须遵守以下规则：
 原始字段必须在别名字段创建之前定义。 原始字段不能是对象类型，也不能是另一个别名字段。  可以使用别名字段的搜索 API #  您可以在以下搜索 API 的只读操作中使用别名："
---


# Alias 别名字段类型

别名字段类型为现有字段创建另一个名称。您可以在搜索和字段功能的 API 操作中使用别名字段，但存在一些例外情况。要设置别名，必须在 path 参数中指定原始字段名称。

## 参考代码

```
PUT movies
{
  "mappings" : {
    "properties" : {
      "year" : {
        "type" : "date"
      },
      "release_date" : {
        "type" : "alias",
        "path" : "year"
      }
    }
  }
}
```

## 参数说明

- **path**：指向原始字段的完整路径，包括所有父对象。例如，`parent.child.field_name`。此参数为必填项。

## 别名（Alias）字段

别名（Alias）字段必须遵循以下规则：

1. 一个别名字段只能引用一个原始字段。
2. 在嵌套对象中，别名必须与原始字段位于相同的嵌套层级。

> 要更改别名引用的字段，需要更新映射配置。但请注意，之前存储的 Percolator 查询中的别名仍会继续引用原始字段，不会自动更新为新的字段引用。

## 原始字段

别名的原始字段必须遵守以下规则：

1. 原始字段必须在别名字段创建之前定义。
2. 原始字段不能是对象类型，也不能是另一个别名字段。

## 可以使用别名字段的搜索 API

您可以在以下搜索 API 的只读操作中使用别名：

- 查询 (Queries)
- 排序 (Sorts)
- 聚合 (Aggregations)
- 存储字段 (stored_fields)
- 文档值字段 (docvalue_fields)
- 建议 (Suggestions)
- 高亮显示 (Highlights)
- 访问字段值的脚本 (Scripts)

## 也可以在字段功能 API 操作中使用别名

要在字段功能 API 中使用别名，请将其指定在 fields 参数中。

```
GET movies/_field_caps?fields=release_date
```

## 例外情况

别名不能用于以下场景：

- 写入请求（如更新请求）。
- 多字段（multi-fields）或 copy_to 的目标字段。
- 作为 \_source 参数用于结果过滤。
- 接受字段名称的 API（如词向量 term vectors）。
- terms、more_like_this 和 geo_shape 这类查询不支持别名字段，因为别名字段在检索文档时无法使用。

## 通配符（Wildcards）

在搜索和字段功能的通配符(Wildcards)查询中，原始字段和别名都会与通配符模式匹配。

```
GET movies/_field_caps?fields=release*
```

