---
title: "扁平化字段类型（Flattened）"
date: 0001-01-01
summary: "Flattened 字段类型 #  在 Easysearch 中，您不需要在索引文档之前指定映射。如果您不指定映射，Easysearch 会使用动态映射自动映射文档中的每个字段及其子字段。当您摄取诸如日志之类的文档时，您可能事先不知道每个字段的子字段名称和类型。在这种情况下，动态映射所有新的子字段可能会快速导致&quot;映射爆炸&quot;，其中不断增长的字段数量可能会降低集群的性能。
flattened 字段类型通过将整个 JSON 对象视为字符串来解决这个问题。可以使用标准的点路径表示法访问 JSON 对象中的子字段，但它们不会被索引成单独的字段以供快速查找。
 注意：点表示法（a.b）中的字段名最大长度为 2^24 − 1。
 相关指南（先读这些） #    映射基础  映射模式  Flat 对象字段类型提供以下优势：
 高效读取：获取性能类似于关键字字段。 内存效率：将整个复杂的 JSON 对象存储在一个字段中而不索引其所有子字段，可以减少索引中的字段数量。 空间效率：Easysearch 不会为 flat 对象中的子字段创建倒排索引，从而节省空间。 迁移兼容性：您可以将数据从支持类似 flat 字段的数据库系统迁移到 Easysearch。  当字段及其子字段主要用于读取而不是用作搜索条件时，应将字段映射为 flat 对象，因为子字段不会被索引。当对象具有大量字段或您事先不知道内容时，flat 对象非常有用。
Flat 对象支持带有和不带有点路径表示法的精确匹配查询。有关支持的查询类型的完整列表，请参见支持的查询。
 在文档中搜索特定嵌套字段的值可能效率低下，因为它可能需要对索引进行完整扫描，这可能是一个昂贵的操作。
 Flat 对象不支持：
 特定类型的解析。 数值运算，如数值比较或数值排序。 文本分析。 高亮显示。 使用点表示法(a.b)的子字段聚合。 按子字段过滤。  支持的查询 #  Flat 对象字段类型支持以下查询：
 Term Terms Terms set Prefix Range Match Multi-match Query string Simple query string Exists Wildcard  限制 #  以下限制适用于 Easysearch 中的 flat 对象："
---


# Flattened 字段类型

在 Easysearch 中，您不需要在索引文档之前指定映射。如果您不指定映射，Easysearch 会使用动态映射自动映射文档中的每个字段及其子字段。当您摄取诸如日志之类的文档时，您可能事先不知道每个字段的子字段名称和类型。在这种情况下，动态映射所有新的子字段可能会快速导致"映射爆炸"，其中不断增长的字段数量可能会降低集群的性能。

`flattened` 字段类型通过将整个 JSON 对象视为字符串来解决这个问题。可以使用标准的点路径表示法访问 JSON 对象中的子字段，但它们不会被索引成单独的字段以供快速查找。

> **注意**：点表示法（`a.b`）中的字段名最大长度为 2^24 − 1。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

Flat 对象字段类型提供以下优势：

- 高效读取：获取性能类似于关键字字段。
- 内存效率：将整个复杂的 JSON 对象存储在一个字段中而不索引其所有子字段，可以减少索引中的字段数量。
- 空间效率：Easysearch 不会为 flat 对象中的子字段创建倒排索引，从而节省空间。
- 迁移兼容性：您可以将数据从支持类似 flat 字段的数据库系统迁移到 Easysearch。

当字段及其子字段主要用于读取而不是用作搜索条件时，应将字段映射为 flat 对象，因为子字段不会被索引。当对象具有大量字段或您事先不知道内容时，flat 对象非常有用。

Flat 对象支持带有和不带有点路径表示法的精确匹配查询。有关支持的查询类型的完整列表，请参见支持的查询。

> 在文档中搜索特定嵌套字段的值可能效率低下，因为它可能需要对索引进行完整扫描，这可能是一个昂贵的操作。

Flat 对象不支持：

- 特定类型的解析。
- 数值运算，如数值比较或数值排序。
- 文本分析。
- 高亮显示。
- 使用点表示法(a.b)的子字段聚合。
- 按子字段过滤。

## 支持的查询

Flat 对象字段类型支持以下查询：

- Term
- Terms
- Terms set
- Prefix
- Range
- Match
- Multi-match
- Query string
- Simple query string
- Exists
- Wildcard

## 限制

以下限制适用于 Easysearch 中的 flat 对象：

- Flat 对象不支持开放参数。
- 不支持使用 Painless 脚本和通配符查询来检索子字段的值。

## 使用 flat 对象

以下示例说明怎么将字段映射为 flat 对象、怎么索引带有 flat 对象字段的文档以及在这些文档中怎么去搜索 flat 对象的值。

首先，创建索引的映射，其中 `issue` 的类型为 `flattened`：

```json
PUT /test-index/
{
  "mappings": {
    "properties": {
      "issue": {
        "type": "flattened"
      }
    }
  }
}
```

然后，索引两个带有 flat 对象字段的文档：

```json
PUT /test-index/_doc/1
{
  "issue": {
    "number": "123456",
    "labels": {
      "version": "2.1",
      "backport": [
        "2.0",
        "1.3"
      ],
      "category": {
        "type": "API",
        "level": "enhancement"
      }
    }
  }
}

PUT /test-index/_doc/2
{
  "issue": {
    "number": "123457",
    "labels": {
      "version": "2.2",
      "category": {
        "type": "API",
        "level": "bug"
      }
    }
  }
}
```

要搜索 flat 对象的值，可以使用 GET 或 POST 请求。即使您不知道字段名称，也可以在整个 flat 对象中搜索叶值。例如，以下请求搜索所有标记为 bug 的问题：

```json
GET /test-index/_search
{
  "query": {
    "match": {"issue": "bug"}
  }
}
```

或者，如果您知道要搜索的子字段名称，请使用点表示法提供字段的路径：

```json
GET /test-index/_search
{
  "query": {
    "match": {"issue.labels.category.level": "bug"}
  }
}
```

在这两种情况下，响应都是相同的，并且包含文档 `2`：

```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0303539,
    "hits": [
      {
        "_index": "test-index",
        "_id": "2",
        "_score": 1.0303539,
        "_source": {
          "issue": {
            "number": "123457",
            "labels": {
              "version": "2.2",
              "category": {
                "type": "API",
                "level": "bug"
              }
            }
          }
        }
      }
    ]
  }
}
```

使用前缀查询，您可以搜索所有以 2. 开头的版本的问题：

```json
GET /test-index/_search
{
  "query": {
    "prefix": {"issue.labels.version": "2."}
  }
}
```

使用范围查询，您可以搜索版本 2.0-2.1 的所有问题：

```json
GET /test-index/_search
{
  "query": {
    "range": {
      "issue": {
        "gte": "2.0",
        "lte": "2.1"
      }
    }
  }
}
```

## 将子字段定义为 flat 对象

您可以将 JSON 对象的子字段定义为 flat 对象。例如，使用以下查询将 `issue.labels` 定义为 `flattened`：

```json
PUT /test-index/
{
  "mappings": {
    "properties": {
      "issue": {
        "properties": {
          "number": {
            "type": "double"
          },
          "labels": {
            "type": "flattened"
          }
        }
      }
    }
  }
}
```

因为 issue.number 不是 flat 对象的一部分，所以您可以使用它来聚合和排序文档。

