---
title: "Wrapper 查询"
date: 0001-01-01
summary: "Wrapper 查询 #  wrapper 查询允许您以 Base64 编码的 JSON 格式提交完整的查询。当查询必须嵌入到仅支持字符串值的上下文中时，它非常有用。
仅当需要管理系统约束时才使用此查询。为了提高可读性和可维护性，最好尽可能使用基于 JSON 的标准查询。
相关指南（先读这些） #    Query DSL 基础  专业查询（Specialized queries）  参考样例 #  使用以下映射创建名为 products 的索引：
PUT /products { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; } } } } 索引示例文档：
POST /products/_bulk { &#34;index&#34;: { &#34;_id&#34;: 1 } } { &#34;title&#34;: &#34;Wireless headphones with noise cancellation&#34; } { &#34;index&#34;: { &#34;_id&#34;: 2 } } { &#34;title&#34;: &#34;Bluetooth speaker&#34; } { &#34;index&#34;: { &#34;_id&#34;: 3 } } { &#34;title&#34;: &#34;Over-ear headphones with rich bass&#34; } 以 Base64 格式编码以下查询："
---


# Wrapper 查询

`wrapper` 查询允许您以 Base64 编码的 JSON 格式提交完整的查询。当查询必须嵌入到仅支持字符串值的上下文中时，它非常有用。

仅当需要管理系统约束时才使用此查询。为了提高可读性和可维护性，最好尽可能使用基于 JSON 的标准查询。

## 相关指南（先读这些）

- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [专业查询（Specialized queries）]({{< relref "/docs/features/query-dsl/specialized/_index.md" >}})

## 参考样例

使用以下映射创建名为 `products` 的索引：

```
PUT /products
{
  "mappings": {
    "properties": {
      "title": { "type": "text" }
    }
  }
}
```

索引示例文档：

```
POST /products/_bulk
{ "index": { "_id": 1 } }
{ "title": "Wireless headphones with noise cancellation" }
{ "index": { "_id": 2 } }
{ "title": "Bluetooth speaker" }
{ "index": { "_id": 3 } }
{ "title": "Over-ear headphones with rich bass" }
```

以 Base64 格式编码以下查询：

```
echo -n '{ "match": { "title": "headphones" } }' | base64
```

执行编码的查询：

```
POST /products/_search
{
  "query": {
    "wrapper": {
      "query": "eyAibWF0Y2giOiB7ICJ0aXRsZSI6ICJoZWFkcGhvbmVzIiB9IH0="
    }
  }
}
```

结果包含两个匹配的文档：

```
{
  ...
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.20098841,
    "hits": [
      {
        "_index": "products",
        "_id": "1",
        "_score": 0.20098841,
        "_source": {
          "title": "Wireless headphones with noise cancellation"
        }
      },
      {
        "_index": "products",
        "_id": "3",
        "_score": 0.18459359,
        "_source": {
          "title": "Over-ear headphones with rich bass"
        }
      }
    ]
  }
}
```

