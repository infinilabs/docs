---
title: "纳秒日期字段类型（Date Nanos）"
date: 0001-01-01
summary: "Date nanoseconds 字段类型 #  日期纳秒字段类型与 日期 字段类型类似，它存储一个日期。然而，date 以毫秒分辨率存储日期，而 date_nanos 以纳秒分辨率存储日期。日期以 long 值的形式存储，表示自纪元以来的纳秒数。因此，支持的日期范围大约是 1970-2262 年。
对 date_nanos 字段的查询被转换为对字段值的 long 表示形式的范围查询。然后使用字段上设置的格式将存储的字段和聚合结果转换为字符串。
 date_nano 字段支持 date 支持的所有格式和参数。你可以使用 || 分隔的多种格式。
 对于 date_nanos 字段，你可以使用 strict_date_optional_time_nanos 格式来保留纳秒值。如果你在将字段映射为 date_nanos 时没有指定格式，默认格式是 strict_date_optional_time||epoch_millis，它允许你以 strict_date_optional_time 或 epoch_millis 格式传递值。strict_date_optional_time 格式支持纳秒的日期，但 epoch_millis 格式仅支持毫秒的日期。
示例 #  创建一个具有 strict_date_optional_time_nanos 格式的 date_nanos 类型的 date 字段的映射：
PUT testindex/_mapping { &#34;properties&#34;: { &#34;date&#34;: { &#34;type&#34;: &#34;date_nanos&#34;, &#34;format&#34; : &#34;strict_date_optional_time_nanos&#34; } } } 将两个文档写入到索引中：
PUT testindex/_doc/1 { &#34;date&#34;: &#34;2022-06-15T10:12:52."
---


# Date nanoseconds 字段类型

日期纳秒字段类型与 [日期](./date.md) 字段类型类似，它存储一个日期。然而，`date` 以毫秒分辨率存储日期，而 `date_nanos` 以纳秒分辨率存储日期。日期以 `long` 值的形式存储，表示自纪元以来的纳秒数。因此，支持的日期范围大约是 1970-2262 年。

对 `date_nanos` 字段的查询被转换为对字段值的 `long` 表示形式的范围查询。然后使用字段上设置的格式将存储的字段和聚合结果转换为字符串。

> `date_nano` 字段支持 `date` 支持的所有格式和参数。你可以使用 `||` 分隔的多种格式。

对于 `date_nanos` 字段，你可以使用 `strict_date_optional_time_nanos` 格式来保留纳秒值。如果你在将字段映射为 `date_nanos` 时没有指定格式，默认格式是 `strict_date_optional_time||epoch_millis`，它允许你以 `strict_date_optional_time` 或 `epoch_millis` 格式传递值。`strict_date_optional_time` 格式支持纳秒的日期，但 `epoch_millis` 格式仅支持毫秒的日期。

## 示例

创建一个具有 `strict_date_optional_time_nanos` 格式的 `date_nanos` 类型的 `date` 字段的映射：

```json
PUT testindex/_mapping
{
  "properties": {
      "date": {
        "type": "date_nanos",
        "format" : "strict_date_optional_time_nanos"
      }
    }
}
```

将两个文档写入到索引中：

```json
PUT testindex/_doc/1
{ "date": "2022-06-15T10:12:52.382719622Z" }

PUT testindex/_doc/2
{ "date": "2022-06-15T10:12:52.382719624Z" }
```

你可以使用范围查询来搜索日期范围：

```json
GET testindex/_search
{
  "query": {
    "range": {
      "date": {
        "gte": "2022-06-15T10:12:52.382719621Z",
        "lte": "2022-06-15T10:12:52.382719623Z"
      }
    }
  }
}
```

响应包含日期在指定范围内的文档：

```json
{
  "took": 43,
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
        "_id": "1",
        "_score": 1,
        "_source": {
          "date": "2022-06-15T10:12:52.382719622Z"
        }
      }
    ]
  }
}
```

在查询带有 date_nanos 字段的文档时，你可以使用 fields 或 docvalue_fields：

```json
GET testindex/_search
{
  "fields": ["date"]
}

GET testindex/_search
{
  "docvalue_fields" : [
    {
      "field" : "date"
    }
  ]
}
```

上述任一查询的响应都包含两个已索引的文档：

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
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1,
        "_source": {
          "date": "2022-06-15T10:12:52.382719622Z"
        },
        "fields": {
          "date": ["2022-06-15T10:12:52.382719622Z"]
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 1,
        "_source": {
          "date": "2022-06-15T10:12:52.382719624Z"
        },
        "fields": {
          "date": ["2022-06-15T10:12:52.382719624Z"]
        }
      }
    ]
  }
}
```

你可以按如下方式对 date_nanos 字段进行排序：

```json
GET testindex/_search
{
  "sort": {
    "date": "asc"
  }
}
```

响应包含排序后的文档：

```json
{
  "took": 5,
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
    "max_score": null,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": null,
        "_source": {
          "date": "2022-06-15T10:12:52.382719622Z"
        },
        "sort": [1655287972382719700]
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": null,
        "_source": {
          "date": "2022-06-15T10:12:52.382719624Z"
        },
        "sort": [1655287972382719700]
      }
    ]
  }
}
```

你也可以使用 Painless 脚本来访问字段的纳秒部分：

```json
GET testindex/_search
{
  "script_fields" : {
    "my_field" : {
      "script" : {
        "lang" : "painless",
        "source" : "doc['date'].value.nano"
      }
    }
  }
}
```

响应仅包含字段的纳秒部分：

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
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "testindex",
        "_id": "1",
        "_score": 1,
        "fields": {
          "my_field": [382719622]
        }
      },
      {
        "_index": "testindex",
        "_id": "2",
        "_score": 1,
        "fields": {
          "my_field": [382719624]
        }
      }
    ]
  }
}
```

