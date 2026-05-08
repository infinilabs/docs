---
title: "IP 地址字段类型（IP）"
date: 0001-01-01
summary: "IP 地址字段类型 #  IP 字段类型用于存储 IPv4 或 IPv6 格式的 IP 地址。
 要表示 IP 地址范围，可以使用 IP 范围字段类型
 参考代码 #  创建一个有 IP 地址的 mapping
PUT testindex { &#34;mappings&#34; : { &#34;properties&#34; : { &#34;ip_address&#34; : { &#34;type&#34; : &#34;ip&#34; } } } } 索引一个有 IP 地址的文档
PUT testindex/_doc/1 { &#34;ip_address&#34; : &#34;10.24.34.0&#34; } 查询一个特定 IP 地址的索引
GET testindex/_doc/1 { &#34;query&#34;: { &#34;term&#34;: { &#34;ip_address&#34;: &#34;10.24.34.0&#34; } } } 搜索 IP 地址及其关联的网络掩码 #  您可以使用无类别域间路由 (CIDR) 表示法查询索引中的 IP 地址。在 CIDR 表示法中，通过斜杠 / 分隔 IP 地址和前缀长度（0–32）。例如，前缀长度为 24 表示匹配所有具有相同前 24 位的 IP 地址。"
---


# IP 地址字段类型

IP 字段类型用于存储 IPv4 或 IPv6 格式的 IP 地址。

> 要表示 IP 地址范围，可以使用 IP [范围字段类型](./range-field-type.md)

## 参考代码

创建一个有 IP 地址的 mapping

```
PUT testindex
{
  "mappings" : {
    "properties" :  {
      "ip_address" : {
        "type" : "ip"
      }
    }
  }
}
```

索引一个有 IP 地址的文档

```
PUT testindex/_doc/1
{
  "ip_address" : "10.24.34.0"
}
```

查询一个特定 IP 地址的索引

```
GET testindex/_doc/1
{
  "query": {
    "term": {
      "ip_address": "10.24.34.0"
    }
  }
}
```

## 搜索 IP 地址及其关联的网络掩码

您可以使用无类别域间路由 (CIDR) 表示法查询索引中的 IP 地址。在 [CIDR 表示法](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation)中，通过斜杠 / 分隔 IP 地址和前缀长度（0–32）。例如，前缀长度为 24 表示匹配所有具有相同前 24 位的 IP 地址。

### 查询 IPV4 格式的参考代码

```
GET testindex/_search
{
  "query": {
    "term": {
      "ip_address": "10.24.34.0/24"
    }
  }
}
```

### 查询 IPV6 格式的参考代码

```
GET testindex/_search
{
  "query": {
    "term": {
      "ip_address": "2001:DB8::/24"
    }
  }
}
```

如果在 query_string 查询中使用 IPv6 格式的 IP 地址，需要对 : 字符进行转义，因为它们会被解析为特殊字符。可以通过将 IP 地址用引号括起来，并使用 \ 转义引号来实现。

```
GET testindex/_search
{
  "query" : {
    "query_string": {
      "query": "ip_address:\"2001:DB8::/24\""
    }
  }
}
```

## 参数说明

下面是 IP 字段的参数说明，都是可选参数
**参数说明：**

| 参数                 | 描述                                                                             | 默认值 |
| -------------------- | -------------------------------------------------------------------------------- | ------ |
| **boost**            | 浮点值，指定字段对相关性评分的权重。大于 1.0 提高相关性，0.0 到 1.0 降低相关性。 | 1.0    |
| **doc_values**       | 布尔值，指定是否将字段存储到磁盘，以便用于聚合、排序或 script 操作。             | true   |
| **ignore_malformed** | 布尔值，指定是否忽略格式错误的值而不抛出异常。                                   | false  |
| **index**            | 布尔值，指定字段是否可被搜索。                                                   | true   |
| **null_value**       | 指定替代 `null` 的值，必须与字段类型一致。如果未指定，值为 `null` 时视为缺失。   | null   |
| **store**            | 布尔值，指定字段值是否单独存储，并可在 `_source` 字段外被检索。                  | false  |

