---
title: "父子关系字段类型（Join）"
date: 0001-01-01
summary: "Join 字段类型 #  join 字段类型用于在同一索引中的文档之间建立父/子关系。
相关指南（先读这些） #    Parent-Child 建模  映射模式  代码样例 #  模拟创建一个映射来建立一个产品和其品牌之间的父/子关系：
PUT testindex1 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;product_to_brand&#34;: { &#34;type&#34;: &#34;join&#34;, &#34;relations&#34;: { &#34;brand&#34;: &#34;product&#34; } } } } } 索引一个父文档：
PUT testindex1/_doc/1 { &#34;name&#34;: &#34;Brand 1&#34;, &#34;product_to_brand&#34;: { &#34;name&#34;: &#34;brand&#34; } } 您也可以使用更简单的格式：
PUT testindex1/_doc/1 { &#34;name&#34;: &#34;Brand 1&#34;, &#34;product_to_brand&#34;: &#34;brand&#34; }  路由要求：在索引子文档时，您需要指定 routing 查询参数，因为同一父/子层级中的父文档和子文档必须索引在同一分片上。每个子文档在 parent 字段中引用其父文档的 ID。更多路由与性能考虑，请参考 Parent-Child 建模章节。"
---


# Join 字段类型

`join` 字段类型用于在同一索引中的文档之间建立父/子关系。

## 相关指南（先读这些）

- [Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

## 代码样例

模拟创建一个映射来建立一个产品和其品牌之间的父/子关系：

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "product_to_brand": {
        "type": "join",
        "relations": {
          "brand": "product"
        }
      }
    }
  }
}
```

索引一个父文档：

```json
PUT testindex1/_doc/1
{
  "name": "Brand 1",
  "product_to_brand": {
    "name": "brand"
  }
}
```

您也可以使用更简单的格式：

```json
PUT testindex1/_doc/1
{
  "name": "Brand 1",
  "product_to_brand": "brand"
}
```

> **路由要求**：在索引子文档时，您需要指定 `routing` 查询参数，因为同一父/子层级中的父文档和子文档必须索引在同一分片上。每个子文档在 `parent` 字段中引用其父文档的 ID。更多路由与性能考虑，请参考[Parent-Child 建模]({{< relref "/docs/best-practices/data-modeling/parent-child.md" >}})章节。

为每个父文档索引两个子文档：

```json
PUT testindex1/_doc/3?routing=1
{
  "name": "Product 1",
  "product_to_brand": {
    "name": "product",
    "parent": "1"
  }
}
```

```json
PUT testindex1/_doc/4?routing=1
{
  "name": "Product 2",
  "product_to_brand": {
    "name": "product",
    "parent": "1"
  }
}
```

## 查询 join 字段

当您查询 join 字段时，返回内容中包含指定返回文档是父文档还是子文档的子字段。对于子对象，还会返回父文档 ID。

### 搜索所有文档

```json
GET testindex1/_search
{
  "query": {
    "match_all": {}
  }
}
```

返回内容：

```json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "testindex1",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "name": "Brand 1",
          "product_to_brand": {
            "name": "brand"
          }
        }
      },
      {
        "_index": "testindex1",
        "_type": "_doc",
        "_id": "3",
        "_score": 1.0,
        "_routing": "1",
        "_source": {
          "name": "Product 1",
          "product_to_brand": {
            "name": "product",
            "parent": "1"
          }
        }
      },
      {
        "_index": "testindex1",
        "_type": "_doc",
        "_id": "4",
        "_score": 1.0,
        "_routing": "1",
        "_source": {
          "name": "Product 2",
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

### 搜索父文档的所有子文档

查找与 Brand 1 相关的所有产品：

```json
GET testindex1/_search
{
  "query": {
    "has_parent": {
      "parent_type": "brand",
      "query": {
        "match": {
          "name": "Brand 1"
        }
      }
    }
  }
}
```

返回内容：

```json
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "testindex1",
        "_type": "_doc",
        "_id": "3",
        "_score": 1.0,
        "_routing": "1",
        "_source": {
          "name": "Product 1",
          "product_to_brand": {
            "name": "product",
            "parent": "1"
          }
        }
      },
      {
        "_index": "testindex1",
        "_type": "_doc",
        "_id": "4",
        "_score": 1.0,
        "_routing": "1",
        "_source": {
          "name": "Product 2",
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

### 搜索子文档的父文档

查找 Product 1 的父文档：

```json
GET testindex1/_search
{
  "query": {
    "has_child": {
      "type": "product",
      "query": {
        "match": {
          "name": "Product 1"
        }
      }
    }
  }
}
```

返回内容：

```json
{
  "took": 4,
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
    "max_score": 1.0,
    "hits": [
      {
        "_index": "testindex1",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "name": "Brand 1",
          "product_to_brand": {
            "name": "brand"
          }
        }
      }
    ]
  }
}
```

### 具有多个子文档的父文档

一个父文档可以有多个子文档。创建一个具有多个子文档的映射：

```json
PUT testindex1
{
  "mappings": {
    "properties": {
      "parent_to_child": {
        "type": "join",
        "relations": {
          "parent": ["child 1", "child 2"]
        }
      }
    }
  }
}
```

### Join 字段类型注意事项

- 一个索引中只能有一个 join 字段映射。

- 在检索、更新或删除子文档时，您需要提供 routing 参数。这是因为同一关系中的父文档和子文档必须索引在同一分片上。

- 不支持一个子文档有多个父文档。

- 父子查询和聚合可能会很耗费资源。它们的性能取决于需要检索的子文档或父文档的数量。

- 您可以向现有的 join 字段添加新的关系。

