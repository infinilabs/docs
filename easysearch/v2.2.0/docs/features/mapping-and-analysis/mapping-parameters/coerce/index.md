---
title: "强制类型转换参数（Coerce）"
date: 0001-01-01
summary: "Coerce 参数 #  coerce 映射参数控制数据在索引期间如何将其值转换为预期的字段数据类型。此参数让您可以验证数据是否按照预期的字段类型正确格式化和索引。这提高了搜索结果的准确性。
相关指南（先读这些） #    映射基础  映射模式  代码样例 #  以下示例演示如何使用 coerce 映射参数。
启用 coerce 去索引文档 #  PUT products { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;price&#34;: { &#34;type&#34;: &#34;integer&#34;, &#34;coerce&#34;: true } } } } PUT products/_doc/1 { &#34;name&#34;: &#34;Product A&#34;, &#34;price&#34;: &#34;19.99&#34; } 在此示例中，price 字段被定义为 integer 类型，且 coerce 设置为 true。在索引文档时，字符串值 19.99 被强制转换为整数 19。
禁用 coerce 的文档索引 #  PUT orders { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;quantity&#34;: { &#34;type&#34;: &#34;integer&#34;, &#34;coerce&#34;: false } } } } PUT orders/_doc/1 { &#34;item&#34;: &#34;Widget&#34;, &#34;quantity&#34;: &#34;10&#34; } 在此示例中，quantity 字段被定义为 integer 类型，且 coerce 设置为 false。在索引文档时，字符串值 10 不会被强制转换，由于类型不匹配，文档写入会被拒绝。"
---


# Coerce 参数

`coerce` 映射参数控制数据在索引期间如何将其值转换为预期的字段数据类型。此参数让您可以验证数据是否按照预期的字段类型正确格式化和索引。这提高了搜索结果的准确性。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [映射模式]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})

## 代码样例

以下示例演示如何使用 `coerce` 映射参数。

##### 启用 `coerce` 去索引文档

```
PUT products
{
  "mappings": {
    "properties": {
      "price": {
        "type": "integer",
        "coerce": true
      }
    }
  }
}

PUT products/_doc/1
{
  "name": "Product A",
  "price": "19.99"
}
```

在此示例中，`price` 字段被定义为 `integer` 类型，且 `coerce` 设置为 `true`。在索引文档时，字符串值 `19.99` 被强制转换为整数 `19`。

##### 禁用 `coerce` 的文档索引

```
PUT orders
{
  "mappings": {
    "properties": {
      "quantity": {
        "type": "integer",
        "coerce": false
      }
    }
  }
}

PUT orders/_doc/1
{
  "item": "Widget",
  "quantity": "10"
}
```

在此示例中，`quantity` 字段被定义为 `integer` 类型，且 `coerce` 设置为 `false`。在索引文档时，字符串值 `10` 不会被强制转换，由于类型不匹配，文档写入会被拒绝。

##### 设置索引级别`coerce`强制转换设置

```
PUT inventory
{
  "settings": {
    "index.mapping.coerce": false
  },
  "mappings": {
    "properties": {
      "stock_count": {
        "type": "integer",
        "coerce": true
      },
      "sku": {
        "type": "keyword"
      }
    }
  }
}

PUT inventory/_doc/1
{
  "sku": "ABC123",
  "stock_count": "50"
}
```

在此示例中，索引级别的 `index.mapping.coerce` 设置为 `false`，这会禁用索引的强制转换。但是，`stock_count` 字段覆盖了此设置，这个字段可以自动转换。

