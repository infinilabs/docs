---
title: "复制到参数（Copy To）"
date: 0001-01-01
summary: "Copy To 参数 #  copy_to 参数允许您将多个字段的值复制到单个字段中。如果您经常跨多个字段搜索，此参数会很有用，因为这样可以达到搜索一组字段的效果。
只有字段值被复制，而不是分析器产生的词项。原始的 _source 字段保持不变，并且可以使用 copy_to 参数将相同的值复制到多个字段。但是，字段间不支持递归复制；相反，应该直接使用 copy_to 从源字段复制到多个目标字段。
相关指南（先读这些） #    映射基础  多字段搜索  代码样例 #  以下示例使用 copy_to 参数通过产品的名称和描述进行搜索，并将这些值复制到单个字段中：
PUT my-products-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;name&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;copy_to&#34;: &#34;product_info&#34; }, &#34;description&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;copy_to&#34;: &#34;product_info&#34; }, &#34;product_info&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;price&#34;: { &#34;type&#34;: &#34;float&#34; } } } } PUT my-products-index/_doc/1 { &#34;name&#34;: &#34;Wireless Headphones&#34;, &#34;description&#34;: &#34;High-quality wireless headphones with noise cancellation&#34;, &#34;price&#34;: 99."
---


# Copy To 参数

`copy_to` 参数允许您将多个字段的值复制到单个字段中。如果您经常跨多个字段搜索，此参数会很有用，因为这样可以达到搜索一组字段的效果。

只有字段值被复制，而不是分析器产生的词项。原始的 `_source` 字段保持不变，并且可以使用 `copy_to` 参数将相同的值复制到多个字段。但是，字段间不支持递归复制；相反，应该直接使用 `copy_to` 从源字段复制到多个目标字段。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [多字段搜索]({{< relref "/docs/features/fulltext-search/multi-field-search.md" >}})

## 代码样例

以下示例使用 `copy_to` 参数通过产品的名称和描述进行搜索，并将这些值复制到单个字段中：

```
PUT my-products-index
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "copy_to": "product_info"
      },
      "description": {
        "type": "text",
        "copy_to": "product_info"
      },
      "product_info": {
        "type": "text"
      },
      "price": {
        "type": "float"
      }
    }
  }
}

PUT my-products-index/_doc/1
{
  "name": "Wireless Headphones",
  "description": "High-quality wireless headphones with noise cancellation",
  "price": 99.99
}

PUT my-products-index/_doc/2
{
  "name": "Bluetooth Speaker",
  "description": "Portable Bluetooth speaker with long battery life",
  "price": 49.99
}
```

在此示例中，`name` 和 `description` 字段的值被复制到 `product_info` 字段中。现在您可以通过查询 `product_info` 字段来搜索产品，如下所示：

```
GET my-products-index/_search
{
  "query": {
    "match": {
      "product_info": "wireless headphones"
    }
  }
}
```

## 响应

```
{
  "took": 20,
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
    "max_score": 1.9061546,
    "hits": [
      {
        "_index": "my-products-index",
        "_id": "1",
        "_score": 1.9061546,
        "_source": {
          "name": "Wireless Headphones",
          "description": "High-quality wireless headphones with noise cancellation",
          "price": 99.99
        }
      }
    ]
  }
}
```

