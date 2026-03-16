---
title: "嵌套聚合（Nested）"
date: 0001-01-01
summary: "嵌套聚合 #  nested 聚合让你能够对嵌套对象内的字段进行聚合。nested 类型是对象数据类型的特殊版本，它允许对象数组以独立于彼此的方式进行索引，从而可以独立于彼此进行查询。
相关指南（先读这些） #    聚合基础  Nested 建模  聚合场景实践  使用 object 类型，所有数据都存储在同一个文档中，因此搜索匹配可以跨越子文档。例如，想象一个 logs 索引，其中 pages 映射为 object 数据类型：
PUT logs/_doc/0 { &#34;response&#34;: &#34;200&#34;, &#34;pages&#34;: [ { &#34;page&#34;: &#34;landing&#34;, &#34;load_time&#34;: 200 }, { &#34;page&#34;: &#34;blog&#34;, &#34;load_time&#34;: 500 } ] } Easysearch 合并所有看起来像这样的实体关系的子属性：
{ &#34;logs&#34;: { &#34;pages&#34;: [&#34;landing&#34;, &#34;blog&#34;], &#34;load_time&#34;: [&#34;200&#34;, &#34;500&#34;] } } 所以，如果你想要用 pages=landing 和 load_time=500 搜索这个索引，即使 load_time 的 landing 值为 200，这个文档也符合条件。"
---


# 嵌套聚合

`nested` 聚合让你能够对嵌套对象内的字段进行聚合。`nested` 类型是对象数据类型的特殊版本，它允许对象数组以独立于彼此的方式进行索引，从而可以独立于彼此进行查询。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [Nested 建模]({{< relref "/docs/best-practices/data-modeling/nested.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

使用 `object` 类型，所有数据都存储在同一个文档中，因此搜索匹配可以跨越子文档。例如，想象一个 `logs` 索引，其中 `pages` 映射为 `object` 数据类型：

```
PUT logs/_doc/0
{
  "response": "200",
  "pages": [
    {
      "page": "landing",
      "load_time": 200
    },
    {
      "page": "blog",
      "load_time": 500
    }
  ]
}

```

Easysearch 合并所有看起来像这样的实体关系的子属性：

```
{
  "logs": {
    "pages": ["landing", "blog"],
    "load_time": ["200", "500"]
  }
}
```

所以，如果你想要用 `pages=landing` 和 `load_time=500` 搜索这个索引，即使 `load_time` 的 `landing` 值为 200，这个文档也符合条件。

如果你想要确保不会发生这种跨对象匹配，将字段映射为 `nested` 类型：

```
PUT logs
{
  "mappings": {
    "properties": {
      "pages": {
        "type": "nested",
        "properties": {
          "page": { "type": "text" },
          "load_time": { "type": "double" }
        }
      }
    }
  }
}

```

嵌套文档允许你索引相同的 JSON 文档，但会保持你的页面在不同的 Lucene 文档中，使得只有 `pages=landing` 和 `load_time=200` 这样的搜索能返回预期结果。内部上，嵌套对象将数组中的每个对象索引为一个单独的隐藏文档，这意味着每个嵌套对象都可以独立于其他对象进行查询。

您必须指定相对于父级包含嵌套文档的嵌套路径：

```
GET logs/_search
{
  "query": {
    "match": { "response": "200" }
  },
  "aggs": {
    "pages": {
      "nested": {
        "path": "pages"
      },
      "aggs": {
        "min_load_time": { "min": { "field": "pages.load_time" } }
      }
    }
  }
}

```

返回内容

```
...
"aggregations" : {
  "pages" : {
    "doc_count" : 2,
    "min_load_time" : {
      "value" : 200
    }
  }
 }
}
```

