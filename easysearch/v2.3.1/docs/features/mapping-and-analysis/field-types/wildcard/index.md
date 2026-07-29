---
title: "通配符字段类型（Wildcard）"
date: 0001-01-01
summary: "Wildcard 字段类型 #  wildcard 字段是 keyword 字段的一种变体，专为任意子字符串和正则表达式匹配而设计。
当您的内容由&quot;字符串&quot;而非&quot;文本&quot;组成时，应使用 wildcard 字段。示例包括非结构化日志行和计算机代码。
wildcard 字段类型的索引方式与 keyword 字段类型不同。keyword 字段将原始字段值写入索引，而 wildcard 字段类型则将字段值拆分为长度小于或等于 3 的子字符串，并将这些子字符串写入索引。例如，字符串 test 被拆分为 t、te、tes、e、es 和 est 这些子字符串。
在搜索时，将查询模式中所需的子字符串与索引进行匹配以生成候选文档，然后根据查询中的模式对这些文档进行过滤。例如，对于搜索词 test，Easysearch 执行索引搜索 tes AND est。如果搜索词包含少于三个字符，Easysearch 会使用长度为一或二的字符子字符串。对于每个匹配的文档，如果源值为 test，则该文档将出现在结果中。这样可以排除误报值，如 nikola tesla felt alternating current was best。
通常，精确匹配查询（如 term 或 terms 查询）在 wildcard 字段上的表现不如在 keyword 字段上有效，而 wildcard、 prefix 和 regexp 查询在 wildcard 字段上表现更好。
相关指南（先读这些） #    映射基础  部分匹配  结构化搜索  示例 #  创建带有 wildcard 字段的映射："
---


# Wildcard 字段类型

`wildcard` 字段是 `keyword` 字段的一种变体，专为任意子字符串和正则表达式匹配而设计。

当您的内容由"字符串"而非"文本"组成时，应使用 `wildcard` 字段。示例包括非结构化日志行和计算机代码。

`wildcard` 字段类型的索引方式与 `keyword` 字段类型不同。`keyword` 字段将原始字段值写入索引，而 `wildcard` 字段类型则将字段值拆分为长度小于或等于 3 的子字符串，并将这些子字符串写入索引。例如，字符串 `test` 被拆分为 `t`、`te`、`tes`、`e`、`es` 和 `est` 这些子字符串。

在搜索时，将查询模式中所需的子字符串与索引进行匹配以生成候选文档，然后根据查询中的模式对这些文档进行过滤。例如，对于搜索词 `test`，Easysearch 执行索引搜索 `tes AND est`。如果搜索词包含少于三个字符，Easysearch 会使用长度为一或二的字符子字符串。对于每个匹配的文档，如果源值为 `test`，则该文档将出现在结果中。这样可以排除误报值，如 `nikola tesla felt alternating current was best`。

通常，精确匹配查询（如 [`term`]({{< relref "/docs/features/query-dsl/term-based-query/term.md" >}}) 或 [`terms`]({{< relref "/docs/features/query-dsl/term-based-query/terms.md" >}}) 查询）在 `wildcard` 字段上的表现不如在 `keyword` 字段上有效，而 [`wildcard`]({{< relref "/docs/features/query-dsl/term-based-query/wildcard.md" >}})、[`prefix`]({{< relref "/docs/features/query-dsl/term-based-query/prefix.md" >}}) 和 [`regexp`]({{< relref "/docs/features/query-dsl/term-based-query/regexp.md" >}}) 查询在 `wildcard` 字段上表现更好。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [部分匹配]({{< relref "/docs/features/fulltext-search/partial-matching.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

## 示例

创建带有 `wildcard` 字段的映射：

```json
PUT logs
{
  "mappings" : {
    "properties" : {
      "log_line" : {
        "type" :  "wildcard"
      }
    }
  }
}
```

## 参数

以下表格列出了 `wildcard` 字段可用的所有参数。   `

参数 | 描述
:--- | :---
`doc_values` | 布尔值，指定该字段是否应存储在磁盘上，以便用于聚合、排序或脚本操作。默认值为 `false`。
`ignore_above` | 长度超过此整数值的任何字符串都不会被索引。默认值为 `2147483647`。
`normalizer` | 用于预处理索引和搜索值的标准化器。默认情况下，不进行标准化，使用原始值。您可以使用 `lowercase` 标准化器在该字段上执行不区分大小写的匹配。
`null_value` | 用于替代 `null` 的值。必须与字段类型相同。如果未指定此参数，则当字段值为 `null` 时，该字段将被视为缺失。默认值为 `null`。
