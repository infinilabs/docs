---
title: "范围字段类型（Range）"
date: 0001-01-01
summary: "Range 字段类型 #  以下表格列出了 Easysearch 支持的所有范围字段类型。
   字段数据类型 描述     integer_range 整数值范围。   long_range 长整型值范围。   double_range 双精度浮点值范围。   float_range 浮点值范围。   ip_range IPv4 或 IPv6 地址范围，起始和结束地址可使用不同格式。   date_range 日期值范围，起始和结束日期可采用不同格式。内部以 64 位无符号整数存储，自纪元以来的毫秒数表示。    相关指南（先读这些） #    映射基础  结构化搜索  参考代码 #  创建一个有双精度浮点数范围字段和日期范围字段的映射
PUT testindex { &#34;mappings&#34; : { &#34;properties&#34; : { &#34;gpa&#34; : { &#34;type&#34; : &#34;double_range&#34; }, &#34;graduation_date&#34; : { &#34;type&#34; : &#34;date_range&#34;, &#34;format&#34; : &#34;strict_year_month||strict_year_month_day&#34; } } } } 索引一个包含这两个字段的文档"
---


# Range 字段类型

以下表格列出了 Easysearch 支持的所有范围字段类型。

| 字段数据类型    | 描述                                                                                                 |
| --------------- | ---------------------------------------------------------------------------------------------------- |
| `integer_range` | 整数值范围。                                                                                          |
| `long_range`    | 长整型值范围。                                                                                        |
| `double_range`  | 双精度浮点值范围。                                                                                    |
| `float_range`   | 浮点值范围。                                                                                          |
| `ip_range`      | IPv4 或 IPv6 地址范围，起始和结束地址可使用不同格式。                                                 |
| `date_range`    | 日期值范围，起始和结束日期可采用不同格式。内部以 64 位无符号整数存储，自纪元以来的毫秒数表示。        |

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

## 参考代码

创建一个有双精度浮点数范围字段和日期范围字段的映射

```
PUT testindex
{
  "mappings" : {
    "properties" :  {
      "gpa" : {
        "type" : "double_range"
      },
      "graduation_date" : {
        "type" : "date_range",
        "format" : "strict_year_month||strict_year_month_day"
      }
    }
  }
}
```

索引一个包含这两个字段的文档

```
PUT testindex/_doc/1
{
  "gpa" : {
    "gte" : 1.0,
    "lte" : 4.0
  },
  "graduation_date" : {
    "gte" : "2019-05-01",
    "lte" : "2019-05-15"
  }
}
```

## IP 地址范围字段

您可以使用两种格式指定 IP 地址范围字段：范围表示法和 [CIDR 表示法](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing#CIDR_notation)。

创建一个有 IP 地址范围字段两个格式的映射

```
PUT testindex
{
  "mappings" : {
    "properties" :  {
      "ip_address_range" : {
        "type" : "ip_range"
      },
      "ip_address_cidr" : {
        "type" : "ip_range"
      }
    }
  }
}
```

索引一个包含这两个 IP 地址范围字段格式的文档

```
PUT testindex/_doc/2
{
  "ip_address_range" : {
    "gte" : "10.24.34.0",
    "lte" : "10.24.35.255"
  },
  "ip_address_cidr" : "10.24.34.0/24"
}
```

## 查询范围字段

您可以使用 Term 查询 或 Range 查询对范围字段进行搜索。

### Term 查询

Term 查询可以使用一个具体值匹配到所有符合条件的范围字段。

例如，以下查询将返回文档 1，因为 3.5 位于文档 1 的范围 [1.0, 4.0] 之内。

```
GET testindex/_search
{
  "query" : {
    "term" : {
      "gpa" : {
        "value" : 3.5
      }
    }
  }
}
```

### 范围查询

对范围字段的范围查询会返回位于指定范围内的文档。

例如，可以查询 2019 年的所有毕业日期，并以“MM/dd/yyyy”格式提供日期范围。

```
GET testindex/_search
{
  "query": {
    "range": {
      "graduation_date": {
        "gte": "01/01/2019",
        "lte": "12/31/2019",
        "format": "MM/dd/yyyy",
        "relation" : "within"
      }
    }
  }
}
```

上面的查询在“within”和“intersects”关系中会返回文档 1，但在“contains”关系中不会返回它。

## 参数详解

下面列出了范围字段的设置参数，所有参数都是可选的
| 参数 | 描述 | 默认值 |  
|-----------|------------------------------------------------------------------------------------------------------|----------|  
| **boost** | 浮点值，指定字段对相关性评分的权重，大于 1.0 提高相关性，0.0 到 1.0 降低相关性。 | 1.0 |  
| **coerce**| 布尔值，允许将小数截断为整数值，并将字符串转换为数值类型。 | true |  
| **index** | 布尔值，指定字段是否可被搜索。 | true |  
| **store** | 布尔值，指定字段值是否单独存储并可从 `_source` 外检索。 | false |

