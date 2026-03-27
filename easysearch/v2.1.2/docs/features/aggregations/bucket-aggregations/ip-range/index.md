---
title: "IP 范围聚合（IP Range）"
date: 0001-01-01
summary: "IP 范围聚合 #  ip_range 聚合用于 IP 地址。它适用于 ip 类型字段。您可以在 CIDR 表示法中定义 IP 范围和掩码。
相关指南（先读这些） #    聚合基础  聚合场景实践  GET sample_data_logs/_search { &#34;size&#34;: 0, &#34;aggs&#34;: { &#34;access&#34;: { &#34;ip_range&#34;: { &#34;field&#34;: &#34;ip&#34;, &#34;ranges&#34;: [ { &#34;from&#34;: &#34;1.0.0.0&#34;, &#34;to&#34;: &#34;126.158.155.183&#34; }, { &#34;mask&#34;: &#34;1.0.0.0/8&#34; } ] } } } } 返回内容
... &#34;aggregations&#34; : { &#34;access&#34; : { &#34;buckets&#34; : [ { &#34;key&#34; : &#34;1.0.0.0/8&#34;, &#34;from&#34; : &#34;1."
---


# IP 范围聚合

`ip_range` 聚合用于 IP 地址。它适用于 `ip` 类型字段。您可以在 CIDR 表示法中定义 IP 范围和掩码。

## 相关指南（先读这些）

- [聚合基础]({{< relref "/docs/fundamentals/aggregations-data-analysis.md" >}})
- [聚合场景实践]({{< relref "/docs/features/aggregations/aggs-recipes.md" >}})

```
GET sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "access": {
      "ip_range": {
        "field": "ip",
        "ranges": [
          {
            "from": "1.0.0.0",
            "to": "126.158.155.183"
          },
          {
            "mask": "1.0.0.0/8"
          }
        ]
      }
    }
  }
}

```

返回内容

```
...
"aggregations" : {
  "access" : {
    "buckets" : [
      {
        "key" : "1.0.0.0/8",
        "from" : "1.0.0.0",
        "to" : "2.0.0.0",
        "doc_count" : 98
      },
      {
        "key" : "1.0.0.0-126.158.155.183",
        "from" : "1.0.0.0",
        "to" : "126.158.155.183",
        "doc_count" : 7184
      }
    ]
  }
 }
}
```

如果您向映射中将 `ip_range` 设置为 `false` 的索引添加了字段格式不正确的文档，Easysearch 会拒绝整个文档。您可以将 `ignore_malformed` 设置为 `true` 以指定 Easysearch 应忽略格式不正确的字段。默认值为 `false` 。

```
...
"mappings": {
  "properties": {
    "ips": {
      "type": "ip_range",
      "ignore_malformed": true
    }
  }
}
```

## 参数说明

| 参数     | 必需/可选 | 数据类型 | 描述                                                                    |
| -------- | --------- | -------- | ----------------------------------------------------------------------- |
| `field`  | 必填      | String   | IP 类型的字段名称。                                                      |
| `ranges` | 必填      | Array    | IP 范围数组。每个元素可含 `from`/`to` 或 `mask`（CIDR 表示法）。          |
| `keyed`  | 可选      | Boolean  | 若为 `true`，以键值对形式返回桶。默认 `false`。                           |

### CIDR 表示法

使用 `mask` 参数可以用 CIDR 表示法定义 IP 范围，常用的网段掩码：

| CIDR       | 范围                         | IP 数量      |
| ---------- | ---------------------------- | ------------ |
| `/8`       | x.0.0.0 - x.255.255.255     | 16,777,216   |
| `/16`      | x.y.0.0 - x.y.255.255       | 65,536       |
| `/24`      | x.y.z.0 - x.y.z.255         | 256          |
| `/32`      | 单个 IP                     | 1            |

> **提示**：`ip_range` 聚合同时支持 IPv4 和 IPv6 地址。

