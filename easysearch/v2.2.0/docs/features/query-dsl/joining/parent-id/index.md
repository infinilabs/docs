---
title: "Parent ID 查询"
date: 0001-01-01
summary: "Parent ID 查询 #  parent_id 查询返回具有指定 ID 的父文档的子文档。您可以通过使用连接字段类型在相同索引中的文档之间建立父子关系。
相关指南（先读这些） #    Parent-Child 建模  关联查询（Joining）  参考样例 #  在您运行一个 parent_id 查询之前，您的索引必须包含一个连接字段，以便建立父子关系。索引映射请求使用以下格式：
PUT /example_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;relationship_field&#34;: { &#34;type&#34;: &#34;join&#34;, &#34;relations&#34;: { &#34;parent_doc&#34;: &#34;child_doc&#34; } } } } } 对于此示例，首先配置一个包含代表产品和其品牌的文档的索引，这些文档在 has_child 查询示例中有所描述。
要搜索特定父文档的子文档，请使用 parent_id 查询。以下查询返回具有 ID 1 的父文档的子文档（产品）：
GET testindex1/_search { &#34;query&#34;: { &#34;parent_id&#34;: { &#34;type&#34;: &#34;product&#34;, &#34;id&#34;: &#34;1&#34; } } } 返回子产品：
{ &#34;took&#34;: 57, &#34;timed_out&#34;: false, &#34;_shards&#34;: { &#34;total&#34;: 1, &#34;successful&#34;: 1, &#34;skipped&#34;: 0, &#34;failed&#34;: 0 }, &#34;hits&#34;: { &#34;total&#34;: { &#34;value&#34;: 1, &#34;relation&#34;: &#34;eq&#34; }, &#34;max_score&#34;: 0."
---


# Parent ID 查询

`parent_id` 查询返回具有指定 ID 的父文档的子文档。您可以通过使用连接字段类型在相同索引中的文档之间建立父子关系。

## 相关指南（先读这些）

- [Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})
- [关联查询（Joining）]({{< relref "/docs/features/query-dsl/joining/_index.md" >}})

## 参考样例

在您运行一个 `parent_id` 查询之前，您的索引必须包含一个连接字段，以便建立父子关系。索引映射请求使用以下格式：

```
PUT /example_index
{
  "mappings": {
    "properties": {
      "relationship_field": {
        "type": "join",
        "relations": {
          "parent_doc": "child_doc"
        }
      }
    }
  }
}
```

对于此示例，首先配置一个包含代表产品和其品牌的文档的索引，这些文档在 `has_child` 查询示例中有所描述。

要搜索特定父文档的子文档，请使用 `parent_id` 查询。以下查询返回具有 ID 1 的父文档的子文档（产品）：

```
GET testindex1/_search
{
  "query": {
    "parent_id": {
      "type": "product",
      "id": "1"
    }
  }
}
```

返回子产品：

```
{
  "took": 57,
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
    "max_score": 0.87546873,
    "hits": [
      {
        "_index": "testindex1",
        "_id": "3",
        "_score": 0.87546873,
        "_routing": "1",
        "_source": {
          "name": "Mechanical watch",
          "sales_count": 150,
          "product_to_brand": {
            "name": "product",
            "parent": "1"
          }
        }
      }
    ]
  }
}
```

## 参数说明

以下表格列出了所有由 `parent_id` 查询支持的一级参数。

| 参数              | 必填/可选 | 描述                                                                                                                                            |
| ----------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`            | 必填      | 指定在 `join` 字段映射中定义的子关系名称。                                                                                                      |
| `id`              | 必填      | 父文档的 ID。查询返回与该父文档关联的子文档。                                                                                                   |
| `ignore_unmapped` | 可选      | 指示是否忽略未映射的 type 字段，而不是抛出错误而不返回文档。您可以在查询多个索引时提供此参数，其中一些索引可能不包含 type 字段。默认为 false 。 |

