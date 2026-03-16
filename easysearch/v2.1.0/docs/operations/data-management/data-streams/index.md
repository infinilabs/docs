---
title: "数据流"
date: 0001-01-01
description: "Data Stream 管理和最佳实践。"
summary: "数据流（Data streams） #  如果你正在将连续生成的时间序列数据（如日志、事件和指标）摄入 Easysearch，那么你很可能处于这样一种场景：文档数量快速增长，且你无需更新旧文档。
管理时间序列数据的典型工作流程包含多个步骤，例如创建滚动索引别名、定义写入索引，以及为底层索引定义通用的映射和设置。
数据流简化了这一过程，并强制采用最适合时间序列数据的配置方式，例如主要为仅追加（append-only）数据设计，并确保每个文档都包含一个时间戳字段。
数据流在内部由多个底层索引组成。搜索请求会被路由到所有底层索引，而写入请求则被路由到最新的写入索引。通过 索引生命周期管理（ILM） 策略，你可以自动处理索引滚动（rollover）或删除操作。
数据流使用说明 #  步骤 1：创建索引模板 #  要创建数据流，首先需要创建一个索引模板，用于将一组索引配置为数据流。data_stream 对象表明这是一个数据流，而非普通索引模板。索引模式需与数据流的名称匹配：
PUT _index_template/logs-template-nginx { &#34;index_patterns&#34;: &#34;logs-nginx&#34;, &#34;data_stream&#34;: { }, &#34;priority&#34;: 200, &#34;template&#34;: { &#34;settings&#34;: { &#34;number_of_shards&#34;: 1, &#34;number_of_replicas&#34;: 0 } } } 在此情况下，每个摄入的文档都必须包含一个 @timestamp 字段。
你也可以在 data_stream 对象中自定义时间戳字段名称。此外，你还可以在此处定义索引映射和其他设置，就像为普通索引模板所做的那样。
PUT _index_template/logs-template-nginx { &#34;index_patterns&#34;: &#34;logs-nginx&#34;, &#34;data_stream&#34;: { &#34;timestamp_field&#34;: { &#34;name&#34;: &#34;request_time&#34; } }, &#34;priority&#34;: 200, &#34;template&#34;: { &#34;settings&#34;: { &#34;number_of_shards&#34;: 1, &#34;number_of_replicas&#34;: 0 } } } 在此示例中，logs-nginx 索引会匹配 logs-template-nginx 模板。当存在多个匹配时，Easysearch 会选择优先级更高的模板。"
---


# 数据流（Data streams）

如果你正在将连续生成的时间序列数据（如日志、事件和指标）摄入 Easysearch，那么你很可能处于这样一种场景：文档数量快速增长，且你无需更新旧文档。

管理时间序列数据的典型工作流程包含多个步骤，例如创建滚动索引别名、定义写入索引，以及为底层索引定义通用的映射和设置。

数据流简化了这一过程，并强制采用最适合时间序列数据的配置方式，例如主要为仅追加（append-only）数据设计，并确保每个文档都包含一个时间戳字段。

数据流在内部由多个底层索引组成。搜索请求会被路由到所有底层索引，而写入请求则被路由到最新的写入索引。通过 索引生命周期管理（ILM） 策略，你可以自动处理索引滚动（rollover）或删除操作。

## 数据流使用说明

### 步骤 1：创建索引模板

要创建数据流，首先需要创建一个索引模板，用于将一组索引配置为数据流。`data_stream` 对象表明这是一个数据流，而非普通索引模板。索引模式需与数据流的名称匹配：

```auto
PUT _index_template/logs-template-nginx
{
  "index_patterns": "logs-nginx",
  "data_stream": {
  },
  "priority": 200,
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}
```

在此情况下，每个摄入的文档都必须包含一个 `@timestamp` 字段。

你也可以在 `data_stream` 对象中自定义时间戳字段名称。此外，你还可以在此处定义索引映射和其他设置，就像为普通索引模板所做的那样。

```auto
PUT _index_template/logs-template-nginx
{
  "index_patterns": "logs-nginx",
  "data_stream": {
    "timestamp_field": {
      "name": "request_time"
    }
  },
  "priority": 200,
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}
```

在此示例中，`logs-nginx` 索引会匹配 `logs-template-nginx` 模板。当存在多个匹配时，Easysearch 会选择优先级更高的模板。

### 步骤 2：创建数据流

创建索引模板后，即可创建数据流。

你可以使用数据流 API 显式创建数据流。该 API 将初始化第一个底层索引：

```auto
PUT _data_stream/logs-nginx
```

你也可以直接开始摄入数据，而无需预先创建数据流。

由于我们已有包含 `data_stream` 对象的匹配索引模板，Easysearch 将自动创建数据流：

```auto
POST logs-nginx/_doc
{
  "message": "login attempt failed",
  "@timestamp": "2013-03-01T00:00:00"
}
```

要查看特定数据流的信息：

```auto
GET _data_stream/logs-nginx
```

#### 示例响应

```auto
{
  "data_streams": [
    {
      "name": "logs-nginx",
      "timestamp_field": {
        "name": "@timestamp"
      },
      "indices": [
        {
          "index_name": ".ds-logs-nginx-000001",
          "index_uuid": "Z8RATqnzTbGxbm-khb5LKw"
        },
        {
          "index_name": ".ds-logs-nginx-000002",
          "index_uuid": "o8xka3CxSjqzNrw1BoRP6A"
        }
      ],
      "generation": 2,
      "status": "GREEN",
      "template": "logs-template-nginx"
    }
  ]
}
```

你可以看到时间戳字段的名称、底层索引列表、用于创建数据流的模板，以及数据流的健康状态（反映其所有底层索引中最低的状态）。

要获取更多关于数据流的详细信息，请使用 `_stats` 端点：

```auto
GET _data_stream/logs-nginx/_stats
```

#### 示例响应

```auto
{
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "data_stream_count": 1,
  "backing_indices": 2,
  "total_store_size_bytes": 18125,
  "data_streams": [
    {
      "data_stream": "logs-nginx",
      "backing_indices": 2,
      "store_size_bytes": 18125,
      "maximum_timestamp": 1362096000000
    }
  ]
}
```

要查看所有数据流的信息，请使用以下请求：

```auto
GET _data_stream
```

### 步骤 3：向数据流中摄入数据

你可以使用常规的索引 API 将数据摄入数据流。请确保每个索引的文档都包含时间戳字段。如果尝试摄入没有时间戳字段的文档，将会报错。

```auto
POST logs-nginx/_doc
{
  "message": "login attempt",
  "@timestamp": "2013-03-01T00:00:00"
}
```
### 步骤 4：搜索数据流

你可以像搜索普通索引或索引别名一样搜索数据流。搜索操作将应用于所有底层索引（即数据流中的全部数据）。

```auto
GET logs-nginx/_search
{
  "query": {
    "match": {
      "message": "login"
    }
  }
}
```

#### 示例响应

```auto
{
  "took" : 514,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : ".ds-logs-redis-000001",
        "_type" : "_doc",
        "_id" : "-rhVmXoBL6BAVWH3mMpC",
        "_score" : 0.2876821,
        "_source" : {
          "message" : "login attempt",
          "@timestamp" : "2013-03-01T00:00:00"
        }
      }
    ]
  }
}
```

### 步骤 5：滚动数据流

滚动操作会创建一个新的底层索引，并将其设置为数据流的新写入索引。

要对数据流执行手动滚动操作：

```auto
POST logs-nginx/_rollover
```

#### 示例响应

```auto
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "old_index": ".ds-logs-nginx-000001",
  "new_index": ".ds-logs-nginx-000002",
  "rolled_over": true,
  "dry_run": false,
  "conditions": {}
}
```

现在，如果你对 `logs-nginx` 数据流执行 `GET` 操作，你会看到其生成（generation） ID 从 1 增加到了 2。

你还可以设置一个 [索引生命周期管理（ILM）策略]({{< relref "/docs/features/data-retention/ilm" >}}) 来自动执行数据流的滚动操作。ILM 策略在底层索引创建时即被应用。当你将策略关联到数据流时，它仅影响该数据流未来的底层索引。

此外，你无需提供 `rollover_alias` 设置，因为 ILM 策略会从底层索引中自动推断此信息。

### 步骤 6：删除数据流

删除操作会自动删除数据流的所有底层索引，然后删除数据流本身。

要删除一个数据流及其所有隐藏的底层索引：

```auto
DELETE _data_stream/<data_stream_name>
```

你可以使用通配符来删除多个数据流。

我们建议使用 ILM 策略来删除数据流中的数据。

---

## API 参考汇总

| 操作 | 方法 | 端点 |
|------|------|------|
| 创建数据流 | `PUT` | `/_data_stream/{name}` |
| 查看数据流 | `GET` | `/_data_stream` 或 `/_data_stream/{name}` |
| 数据流统计 | `GET` | `/_data_stream/_stats` 或 `/_data_stream/{name}/_stats` |
| 删除数据流 | `DELETE` | `/_data_stream/{name}` |
| 滚动数据流 | `POST` | `/{data_stream}/_rollover` |

> 删除操作支持逗号分隔的列表和通配符，如 `DELETE /_data_stream/logs-*`。

---

## 相关文档

- [索引模板]({{< relref "./index-templates" >}})：数据流依赖索引模板定义
- [索引生命周期管理]({{< relref "/docs/features/data-retention/ilm" >}})：自动管理数据流的滚动与删除
- [数据生命周期]({{< relref "/docs/features/data-retention/lifecycle" >}})：完整的数据保留策略体系
