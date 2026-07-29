---
title: "索引名称元数据字段（_index）"
date: 0001-01-01
summary: "_index 元数据字段 #  当跨多个索引进行查询时，您可能需要根据文档所在的索引来过滤结果。_index 字段根据文档的索引来匹配文档。
相关指南（先读这些） #    映射基础  元数据字段  以下示例创建两个索引，products 和 customers，并向每个索引添加一个文档：
PUT products/_doc/1 { &#34;name&#34;: &#34;Widget X&#34; } PUT customers/_doc/2 { &#34;name&#34;: &#34;John Doe&#34; } 然后，您可以查询这两个索引，并使用 _index 属性过滤结果，如以下示例请求所示：
GET products,customers/_search { &#34;query&#34;: { &#34;terms&#34;: { &#34;_index&#34;: [&#34;products&#34;, &#34;customers&#34;] } }, &#34;aggs&#34;: { &#34;index_groups&#34;: { &#34;terms&#34;: { &#34;field&#34;: &#34;_index&#34;, &#34;size&#34;: 10 } } }, &#34;sort&#34;: [ { &#34;_index&#34;: { &#34;order&#34;: &#34;desc&#34; } } ], &#34;script_fields&#34;: { &#34;index_name&#34;: { &#34;script&#34;: { &#34;lang&#34;: &#34;painless&#34;, &#34;source&#34;: &#34;doc[&#39;_index&#39;]."
---


# _index 元数据字段

当跨多个索引进行查询时，您可能需要根据文档所在的索引来过滤结果。`_index` 字段根据文档的索引来匹配文档。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [元数据字段]({{< relref "/docs/features/mapping-and-analysis/metadata-field/_index.md" >}})

以下示例创建两个索引，`products` 和 `customers`，并向每个索引添加一个文档：

```
PUT products/_doc/1
{
  "name": "Widget X"
}

PUT customers/_doc/2
{
  "name": "John Doe"
}
```

然后，您可以查询这两个索引，并使用 `_index` 属性过滤结果，如以下示例请求所示：

```
GET products,customers/_search
{
  "query": {
    "terms": {
      "_index": ["products", "customers"]
    }
  },
  "aggs": {
    "index_groups": {
      "terms": {
        "field": "_index",
        "size": 10
      }
    }
  },
  "sort": [
    {
      "_index": {
        "order": "desc"
      }
    }
  ],
  "script_fields": {
    "index_name": {
      "script": {
        "lang": "painless",
        "source": "doc['_index'].value"
      }
    }
  }
}
```

在此示例中：

- `query` 部分使用 `terms` 查询来匹配来自 `products` 和 `customers` 索引的文档。
- `aggs` 部分对 `_index` 字段执行 `terms` 聚合，按索引对结果进行分组。
- `sort` 部分按 `_index` 字段以降序对结果进行排序。
- `script_fields` 部分向搜索结果添加一个名为 `index_name` 的新字段，其中包含每个文档的 `_index` 字段值。

## 在 `_index` 字段上查询

`_index` 字段表示文档所在的索引。您可以在查询中使用此字段来过滤、聚合、排序或检索搜索结果的索引信息。

由于 `_index` 字段会自动添加到每个文档中，您可以像使用其他字段一样在查询中使用它。例如，您可以使用 `terms` 查询来匹配来自多个索引的文档。以下示例查询返回来自 `products` 和 `customers` 索引的所有文档：

```
{
  "query": {
    "terms": {
      "_index": ["products", "customers"]
    }
  }
}
```

