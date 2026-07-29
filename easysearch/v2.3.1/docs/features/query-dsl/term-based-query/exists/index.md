---
title: "Exists 查询"
date: 0001-01-01
summary: "Exists 查询 #  使用 exists 查询来搜索包含特定字段的文档。
相关指南（先读这些） #    结构化搜索  Query DSL 基础  如果出现以下任一情况，索引值将不会存在于文档字段中：
 该字段在映射中指定了 &quot;index&quot; : false 。 源 JSON 中的字段为 null 或 [] 。 字段值的长度超过了映射中 ignore_above 的设置。 字段值格式错误，并且映射中定义了 ignore_malformed 。  索引值将在以下情况下存在于文档字段中：
 该值是一个包含一个或多个 null 元素和一个或多个非 null 元素的数组（例如， [&quot;one&quot;, null] ）。 该值是一个空字符串（ &quot;&quot; 或 &quot;-&quot; ）。 该值是一个自定义的 null_value ，如字段映射中所定义。  参考样例 #  例如，假设索引包含以下两个文档：
PUT testindex/_doc/1 { &#34;title&#34;: &#34;The wind rises&#34; } PUT testindex/_doc/2 { &#34;title&#34;: &#34;Gone with the wind&#34;, &#34;description&#34;: &#34;A 1939 American epic historical film&#34; } 以下查询搜索包含 description 字段的文档："
---


# Exists 查询

使用 `exists` 查询来搜索包含特定字段的文档。

## 相关指南（先读这些）

- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})

如果出现以下任一情况，索引值将不会存在于文档字段中：

- 该字段在映射中指定了 `"index" : false` 。
- 源 JSON 中的字段为 `null` 或 `[]` 。
- 字段值的长度超过了映射中 `ignore_above` 的设置。
- 字段值格式错误，并且映射中定义了 `ignore_malformed` 。

索引值将在以下情况下存在于文档字段中：

- 该值是一个包含一个或多个 null 元素和一个或多个非 null 元素的数组（例如， `["one", null]` ）。
- 该值是一个空字符串（ `""` 或 `"-"` ）。
- 该值是一个自定义的 `null_value` ，如字段映射中所定义。

## 参考样例

例如，假设索引包含以下两个文档：

```
PUT testindex/_doc/1
{
  "title": "The wind rises"
}

PUT testindex/_doc/2
{
  "title": "Gone with the wind",
  "description": "A 1939 American epic historical film"
}
```

以下查询搜索包含 `description` 字段的文档：

```
GET testindex/_search
{
  "query": {
    "exists": {
      "field": "description"
    }
  }
}
```

返回内容包含匹配的文档：

```
{
  "took": 3,
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
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "Gone with the wind",
          "description": "A 1939 American epic historical film"
        }
      }
    ]
  }
}
```

## 查找索引值缺失的文档

要查找缺少索引值的文档，可以使用 `must_not` 布尔查询结合内部 `exists` 查询。例如，以下请求将查找 `description` 字段缺失的文档：

```
GET testindex/_search
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "description"
        }
      }
    }
  }
}

```

返回内容包含匹配的文档：

```
{
  "took": 19,
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
    "max_score": 0,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 0,
        "_source": {
          "title": "The wind rises"
        }
      }
    ]
  }
}
```

## 参数说明

| 参数 | 数据类型 | 描述 |
| --- | --- | --- |
| `field` | String | 要检查是否存在的字段名称。必填。 |
| `boost` | Float | 一个浮点数，用于指定该字段对相关性评分的权重。值大于 1.0 会增加字段的相关性。值在 0.0 到 1.0 之间会降低字段的相关性。默认值为 1.0。 |

