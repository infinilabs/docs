---
title: "布尔字段类型（Boolean）"
date: 0001-01-01
summary: "布尔字段类型 #  布尔字段类型接受 true 或 false 值，也支持字符串形式的 &ldquo;true&rdquo; 或 &ldquo;false&rdquo;。此外，还可以使用空字符串 &quot;&quot; 表示 false 值。
参考代码 #  创建一个由 a,b,c 三个布尔字段组成的 mapping
PUT testindex { &#34;mappings&#34; : { &#34;properties&#34; : { &#34;a&#34; : { &#34;type&#34; : &#34;boolean&#34; }, &#34;b&#34; : { &#34;type&#34; : &#34;boolean&#34; }, &#34;c&#34; : { &#34;type&#34; : &#34;boolean&#34; } } } } 索引由布尔值组成的文档
PUT testindex/_doc/1 { &#34;a&#34; : true, &#34;b&#34; : &#34;true&#34;, &#34;c&#34; : &#34;&#34; } 因此，字段 a 和 b 将被设置为 true，而字段 c 将被设置为 false。"
---


# 布尔字段类型

布尔字段类型接受 true 或 false 值，也支持字符串形式的 "true" 或 "false"。此外，还可以使用空字符串 "" 表示 false 值。

## 参考代码

创建一个由 a,b,c 三个布尔字段组成的 mapping

```
PUT testindex
{
  "mappings" : {
    "properties" :  {
      "a" : {
        "type" : "boolean"
      },
      "b" : {
        "type" : "boolean"
      },
      "c" : {
        "type" : "boolean"
      }
    }
  }
}
```

索引由布尔值组成的文档

```
PUT testindex/_doc/1
{
  "a" : true,
  "b" : "true",
  "c" : ""
}
```

因此，字段 a 和 b 将被设置为 true，而字段 c 将被设置为 false。

要搜索所有 c 为 false 的文档，可以使用以下查询示例：

```
GET testindex/_search
{
  "query": {
      "term" : {
        "c" : false
    }
  }
}
```

## 参数说明

下表列出了布尔字段类型支持的参数，所有参数均为可选项。
| 参数 | 描述 | 默认值 |  
|-------------|------------------------------------------------------------------------------------------------------------------------|-----------|  
| **boost** | 浮点值，指定字段对相关性评分的权重，大于 1.0 提高相关性，0.0 到 1.0 降低相关性。 | 1.0 |  
| **doc_values** | 布尔值，指定是否将字段存储到磁盘以供聚合、排序或脚本操作使用。 | true |  
| **index** | 布尔值，指定字段是否可被搜索。 | true |  
| **meta** | _接受字段的元数据_。 | 无 |  
| **null_value** | 替代 `null` 的值，类型必须与字段一致。如果未指定，当值为 `null` 时视为缺失。 | null |  
| **store** | 布尔值，指定字段值是否单独存储并可从 `_source` 外检索。 | false |

## 在聚合和 script 中使用布尔值

在布尔字段的聚合操作中：key 返回数值，true 对应 1，false 对应 0；key_as_string 返回字符串表示，"true" 或 "false"。Script 操作直接返回布尔值 true 或 false。

### 参考代码

运行一个 term 聚合在字段 a 上面

```
GET testindex/_search
{
  "aggs": {
    "agg1": {
      "terms": {
        "field": "a"
      }
    }
  },
  "script_fields": {
    "a": {
      "script": {
        "lang": "painless",
        "source": "doc['a'].value"
      }
    }
  }
}
```

Script 将字段 a 的值返回为 true，key 将其值返回为数值 1，而 key_as_string 则返回字符串 "true"。

```
{
  "took" : 1133,
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "testindex",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "fields" : {
          "a" : [
            true
          ]
        }
      }
    ]
  },
  "aggregations" : {
    "agg1" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 1,
          "key_as_string" : "true",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

