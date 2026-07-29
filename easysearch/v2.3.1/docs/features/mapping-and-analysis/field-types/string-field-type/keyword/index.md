---
title: "关键字字段类型（Keyword）"
date: 0001-01-01
summary: "Keyword 字段类型 #  keyword 字段类型包含未经分析的字符串。它只允许精确的大小写敏感匹配。
默认情况下，keyword 字段既被索引（因为 index 已启用）也存储在磁盘上（因为 doc_values 已启用）。为了减少磁盘空间，您可以通过将 index 设置为 false 来指定不索引 keyword 字段。
 提示：如果您需要对字段进行全文搜索，请将其映射为 text 类型。
 相关指南（先读这些） #    映射基础  映射模式  结构化搜索  代码样例 #  以下查询创建了一个带有 keyword 字段的映射。将 index 设置为 false 指定将 genre 字段存储在磁盘上，并使用 doc_values 检索它：
PUT movies { &#34;mappings&#34; : { &#34;properties&#34; : { &#34;genre&#34; : { &#34;type&#34; : &#34;keyword&#34;, &#34;index&#34; : false } } } } 参数说明 #  下表列出了 keyword 字段类型接受的参数。所有参数都是可选的。"
---


# Keyword 字段类型

`keyword` 字段类型包含未经分析的字符串。它只允许精确的大小写敏感匹配。

默认情况下，`keyword` 字段既被索引（因为 `index` 已启用）也存储在磁盘上（因为 `doc_values` 已启用）。为了减少磁盘空间，您可以通过将 `index` 设置为 `false` 来指定不索引 `keyword` 字段。

> **提示**：如果您需要对字段进行全文搜索，请将其映射为 `text` 类型。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

## 代码样例

以下查询创建了一个带有 keyword 字段的映射。将 `index` 设置为 `false` 指定将 `genre` 字段存储在磁盘上，并使用 `doc_values` 检索它：

```json
PUT movies
{
  "mappings" : {
    "properties" : {
      "genre" : {
        "type" : "keyword",
        "index" : false
      }
    }
  }
}
```

## 参数说明

下表列出了 keyword 字段类型接受的参数。所有参数都是可选的。

| 参数                        | 描述                                                                                                                                         |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| boost                       | 浮点值，指定此字段对相关性分数的权重。大于 1.0 的值会增加字段的相关性。0.0 到 1.0 之间的值会降低字段的相关性。默认值为 1.0。                 |
| doc_values                  | 布尔值，指定是否应将字段存储在磁盘上，以便可以用于聚合、排序或脚本。默认值为 true。                                                          |
| eager_global_ordinals       | 指定是否应在刷新时立即加载全局序号。如果该字段经常用于聚合，此参数应设置为 true。默认值为 false。                                            |
| fields                      | 要以多种方式索引同一个字符串（例如，作为 keyword 和 text），请提供 fields 参数。您可以指定字段的一个版本用于搜索，另一个版本用于排序和聚合。 |
| ignore_above                | 任何长度超过此整数值的字符串都不应被索引。默认值为 2147483647。默认动态映射会创建一个 ignore_above 设置为 256 的 keyword 子字段。            |
| index                       | 布尔值，指定字段是否应可搜索。默认值为 true。要减少磁盘空间，请将 index 设置为 false。                                                       |
| index_options               | 要存储在索引中的信息，将在计算相关性分数时考虑。可以设置为 freqs 以获取词频。默认值为 docs。                                                 |
| meta                        | 接受此字段的元数据。                                                                                                                         |
| normalizer                  | 指定在索引之前如何预处理此字段（例如，转换为小写）。默认值为 null（无预处理）。                                                              |
| norms                       | 布尔值，指定在计算相关性分数时是否应使用字段长度。默认值为 false。                                                                           |
| null_value                  | 用于替代 null 的值。必须与字段类型相同。如果未指定此参数，当值为 null 时，该字段将被视为缺失。默认值为 null。                                |
| similarity                  | 用于计算相关性分数的排名算法。默认值为 BM25。                                                                                                |
| split_queries_on_whitespace | 布尔值，指定全文查询是否应在空格上分割。默认值为 false。                                                                                     |
| store                       | 布尔值，指定字段值是否应该被存储并且可以与 \_source 字段分开检索。默认值为 false。                                                           |

## 典型用法

### 1. 精确匹配查询

keyword 字段最常见的用途是精确匹配，配合 `term` 或 `terms` 查询使用：

```json
GET my-index/_search
{
  "query": {
    "term": {
      "status": "published"
    }
  }
}
```

### 2. 聚合与排序

keyword 字段天然支持聚合和排序（通过 `doc_values`），无需额外配置：

```json
GET my-index/_search
{
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category",
        "size": 20
      }
    }
  }
}
```

### 3. text + keyword 多字段模式

最常见的实战模式是让同一个字段同时支持全文搜索和精确匹配/聚合：

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
```

- 搜索时使用 `title` 字段（全文分词匹配）
- 聚合/排序时使用 `title.keyword` 字段（精确值）

> 动态映射默认就会为字符串创建这种 `text` + `keyword` 多字段结构。

### 4. 使用 normalizer 统一大小写

如果需要忽略大小写进行精确匹配，可以配置 `normalizer`：

```json
PUT my-index
{
  "settings": {
    "analysis": {
      "normalizer": {
        "lowercase_normalizer": {
          "type": "custom",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "email": {
        "type": "keyword",
        "normalizer": "lowercase_normalizer"
      }
    }
  }
}
```

### 5. 控制 ignore_above

动态映射会默认为 keyword 子字段设置 `ignore_above: 256`。如果需要索引更长的字符串，可以手动设置：

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "description": {
        "type": "keyword",
        "ignore_above": 1024
      }
    }
  }
}
```

> 超过 `ignore_above` 长度的值仍然会存储在 `_source` 中，只是不会被索引和聚合。

## keyword 与 text 的对比

| 特性 | keyword | text |
|------|---------|------|
| 是否分词 | ❌ 不分词 | ✅ 经过分词器处理 |
| 精确匹配 | ✅ 支持 | ❌ 不支持（分词后无法精确匹配原始值） |
| 全文搜索 | ❌ 不支持 | ✅ 支持 |
| 排序/聚合 | ✅ 默认支持（doc_values） | ⚠️ 需要启用 fielddata（不推荐） |
| 适用场景 | 状态码、标签、ID、邮箱 | 文章标题、正文、描述 |

## 注意事项

- keyword 字段的值**区分大小写**，如需忽略大小写请使用 `normalizer`
- 对于超高基数字段（如用户 ID、UUID），聚合时注意内存消耗
- 如果字段仅用于过滤而不需要评分，建议设置 `norms: false`（keyword 默认已是 false）以节省空间

