---
title: "渗滤器字段类型（Percolator）"
date: 0001-01-01
summary: "Percolator 字段类型 #  percolator 字段类型将该字段视为查询处理。任何 JSON 对象字段都可以标记为 percolator 字段。通常，文档被索引并用于搜索，而 percolator 字段存储搜索条件，稍后通过 Percolate 查询将匹配文档到该条件。
相关指南（先读这些） #    映射基础  渗透查询  参考代码 #  客户正在搜索价格在 400 美元或以下的桌子，并希望为此搜索创建警报。 创建一个映射，为查询字段分配一个 percolator 字段类型：
PUT testindex1 { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;search&#34;: { &#34;properties&#34;: { &#34;query&#34;: { &#34;type&#34;: &#34;percolator&#34; } } }, &#34;price&#34;: { &#34;type&#34;: &#34;float&#34; }, &#34;item&#34;: { &#34;type&#34;: &#34;text&#34; } } } } 索引一个查询
PUT testindex1/_doc/1 { &#34;search&#34;: { &#34;query&#34;: { &#34;bool&#34;: { &#34;filter&#34;: [ { &#34;match&#34;: { &#34;item&#34;: { &#34;query&#34;: &#34;table&#34; } } }, { &#34;range&#34;: { &#34;price&#34;: { &#34;lte&#34;: 400."
---


# Percolator 字段类型

`percolator` 字段类型将该字段视为查询处理。任何 JSON 对象字段都可以标记为 `percolator` 字段。通常，文档被索引并用于搜索，而 `percolator` 字段存储搜索条件，稍后通过 Percolate 查询将匹配文档到该条件。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [渗透查询]({{< relref "/docs/features/query-dsl/specialized/percolate.md" >}})

## 参考代码

客户正在搜索价格在 400 美元或以下的桌子，并希望为此搜索创建警报。
创建一个映射，为查询字段分配一个 percolator 字段类型：

```
PUT testindex1
{
  "mappings": {
    "properties": {
      "search": {
        "properties": {
          "query": {
            "type": "percolator"
          }
        }
      },
      "price": {
        "type": "float"
      },
      "item": {
        "type": "text"
      }
    }
  }
}
```

索引一个查询

```
PUT testindex1/_doc/1
{
  "search": {
    "query": {
      "bool": {
        "filter": [
          {
            "match": {
              "item": {
                "query": "table"
              }
            }
          },
          {
            "range": {
              "price": {
                "lte": 400.00
              }
            }
          }
        ]
      }
    }
  }
}
```

> 查询中引用的字段必须已经存在于映射中。

运行一个 percolate 查询

```
GET testindex1/_search
{
  "query" : {
    "bool" : {
      "filter" :
        {
          "percolate" : {
            "field" : "search.query",
            "document" : {
              "item" : "Mahogany table",
              "price": 399.99
            }
          }
        }
    }
  }
}
```

返回结果包含原来索引的查询内容

```
{
  "took" : 30,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "testindex1",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.0,
        "_source" : {
          "search" : {
            "query" : {
              "bool" : {
                "filter" : [
                  {
                    "match" : {
                      "item" : {
                        "query" : "table"
                      }
                    }
                  },
                  {
                    "range" : {
                      "price" : {
                        "lte" : 400.0
                      }
                    }
                  }
                ]
              }
            }
          }
        },
        "fields" : {
          "_percolator_document_slot" : [
            0
          ]
        }
      }
    ]
  }
}
```

