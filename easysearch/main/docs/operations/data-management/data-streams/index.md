---
title: "数据流"
date: 0001-01-01
description: "Data Stream 管理和最佳实践。"
summary: "数据流（Data streams） #  如果你正在将连续生成的时间序列数据（如日志、事件和指标）摄入 Easysearch，那么你很可能处于这样一种场景：文档数量快速增长，且你无需更新旧文档。
管理时间序列数据的典型工作流程包含多个步骤，例如创建滚动索引别名、定义写入索引，以及为底层索引定义通用的映射和设置。
数据流简化了这一过程，并强制采用最适合时间序列数据的配置方式，例如主要为仅追加（append-only）数据设计，并确保每个文档都包含一个时间戳字段。
数据流在内部由多个底层索引组成。搜索请求会被路由到所有底层索引，而写入请求则被路由到最新的写入索引。通过 索引生命周期管理（ILM） 策略，你可以自动处理索引滚动（rollover）或删除操作。
数据流使用说明 #  步骤 1：创建索引模板 #  要创建数据流，首先需要创建一个索引模板，用于将一组索引配置为数据流。data_stream 对象表明这是一个数据流，而非普通索引模板。这里的 index_patterns 不是用来匹配底层隐藏索引（例如 .ds-...），而是用来匹配数据流名称本身；也可以把它理解为数据流对外使用的名称：
PUT _index_template/logs-template-nginx { &#34;index_patterns&#34;: &#34;logs-nginx&#34;, &#34;data_stream&#34;: { }, &#34;priority&#34;: 200, &#34;template&#34;: { &#34;settings&#34;: { &#34;number_of_shards&#34;: 1, &#34;number_of_replicas&#34;: 0 } } } 在此情况下，每个摄入的文档都必须包含一个 @timestamp 字段。
你也可以在 data_stream 对象中自定义时间戳字段名称。此外，你还可以在此处定义索引映射和其他设置，就像为普通索引模板所做的那样。
PUT _index_template/logs-template-nginx { &#34;index_patterns&#34;: &#34;logs-nginx&#34;, &#34;data_stream&#34;: { &#34;timestamp_field&#34;: { &#34;name&#34;: &#34;request_time&#34; } }, &#34;priority&#34;: 200, &#34;template&#34;: { &#34;settings&#34;: { &#34;number_of_shards&#34;: 1, &#34;number_of_replicas&#34;: 0 } } } 在此示例中，名称为 logs-nginx 的数据流会匹配 logs-template-nginx 模板，而不是让模板去匹配底层 ."
---


# 数据流（Data streams）

如果你正在将连续生成的时间序列数据（如日志、事件和指标）摄入 Easysearch，那么你很可能处于这样一种场景：文档数量快速增长，且你无需更新旧文档。

管理时间序列数据的典型工作流程包含多个步骤，例如创建滚动索引别名、定义写入索引，以及为底层索引定义通用的映射和设置。

数据流简化了这一过程，并强制采用最适合时间序列数据的配置方式，例如主要为仅追加（append-only）数据设计，并确保每个文档都包含一个时间戳字段。

数据流在内部由多个底层索引组成。搜索请求会被路由到所有底层索引，而写入请求则被路由到最新的写入索引。通过 索引生命周期管理（ILM） 策略，你可以自动处理索引滚动（rollover）或删除操作。

## 数据流使用说明

### 步骤 1：创建索引模板

要创建数据流，首先需要创建一个索引模板，用于将一组索引配置为数据流。`data_stream` 对象表明这是一个数据流，而非普通索引模板。这里的 `index_patterns` 不是用来匹配底层隐藏索引（例如 `.ds-...`），而是用来匹配数据流名称本身；也可以把它理解为数据流对外使用的名称：

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

在此示例中，名称为 `logs-nginx` 的数据流会匹配 `logs-template-nginx` 模板，而不是让模板去匹配底层 `.ds-logs-nginx-*` 隐藏索引。当存在多个匹配时，Easysearch 会选择优先级更高的模板。

### 步骤 2：创建数据流

如果你采用标准的数据流创建流程，前提是已经存在匹配该名称、且带有 `data_stream` 定义的组合模板。常规方式主要有两种：

- `PUT _data_stream/{name}`：显式创建数据流，并初始化第一个底层索引
- 首次写入时自动创建：在第一次写入文档时，自动创建数据流及其首个底层索引，然后完成本次写入

除此之外，Easysearch 还新增了一个复合 API：`PUT _data_stream/{name}/_bootstrap`。它会在缺少匹配模板时自动补一个最小可用模板，然后继续创建数据流。这个接口下面单独说明。

#### `PUT _data_stream/{name}`

标准数据流 API 会初始化第一个底层索引，但前提是已经存在匹配该数据流名称、且带有 `data_stream` 定义的组合模板。

**请求示例**

```auto
PUT _data_stream/logs-nginx
```

这个接口通常无需请求体。数据流的设置、映射以及 `timestamp_field` 等约束都来自已匹配的数据流模板，而不是 `PUT _data_stream/{name}` 的请求体。

如果没有匹配的数据流模板，上面的请求会返回 `400 no matching index template found for data stream [...]`。

#### 首次写入自动创建数据流

如果已经存在包含 `data_stream` 对象的匹配索引模板，你也可以直接开始摄入数据，而无需预先创建数据流。Easysearch 会在第一次写入时自动创建数据流及其首个底层索引。

**请求示例**

如果相关数据流模板（无论是手工创建，还是由 `_bootstrap` 自动创建）把 `timestamp_field` 改成了 `event_time`，后续写入文档时也必须提供 `event_time` 字段，例如：

```auto
POST logs-nginx/_doc
{
  "message": "login attempt failed",
  "event_time": "2013-03-01T00:00:00"
}
```

如果你沿用默认的 `@timestamp`，则可以继续使用下面的写入方式：

```auto
POST logs-nginx/_doc
{
  "message": "login attempt failed",
  "@timestamp": "2013-03-01T00:00:00"
}
```

创建完成后，你可以用下面的请求查看特定数据流的信息：
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
        }
      ],
      "generation": 1,
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
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "data_stream_count": 1,
  "backing_indices": 1,
  "total_store_size_bytes": 4455,
  "data_streams": [
    {
      "data_stream": "logs-nginx",
      "backing_indices": 1,
      "store_size_bytes": 4455,
      "maximum_timestamp": 1362096000000
    }
  ]
}
```

要查看所有数据流的信息，请使用以下请求：

```auto
GET _data_stream
```

#### 推荐使用：新增复合 API `PUT _data_stream/{name}/_bootstrap`

> 推荐场景：前端联调、测试验证、快速起一个可用数据流。

这是 Easysearch 为提升数据流创建体验而新增的复合 API，重点解决“必须先手工准备模板，创建步骤偏多、联调不够顺手”这类问题。

它可以理解为“按需补模板 + 创建数据流”的便捷入口：会先尝试直接创建数据流；仅当缺少匹配模板时，才自动创建一个最小可用的默认模板，然后再次创建数据流。相比标准创建方式，这个入口更适合前端联调、测试验证，以及想快速起一个可用数据流的场景。

如果你是前端开发或测试人员，建议先看下面这条最小联调链路；它覆盖了“创建数据流、查询数据流、查询模板、写入文档、立即搜索验证”这 5 个最常用动作。

##### 前端联调最小链路

推荐为每次联调换一个唯一的数据流名称：

```auto
PUT _data_stream/logs-nginx-ui-001/_bootstrap
GET _data_stream/logs-nginx-ui-001
GET _index_template/ds-bootstrap-logs-nginx-ui-001
POST logs-nginx-ui-001/_doc?refresh=wait_for
{
  "message": "login attempt failed",
  "@timestamp": "2013-03-01T00:00:00"
}
GET logs-nginx-ui-001/_search
```

这条链路分别用于验证：

- `_bootstrap` 是否成功创建数据流
- 响应里的 `template_created` / `template_name` 是否符合预期
- 数据流和自动创建出的模板是否能被查询到
- 前端写入的数据是否能在同一轮联调里立刻被搜索到

其中 `refresh=wait_for` 很重要。若省略它，文档刚写入后立即搜索，可能暂时看不到刚写入的数据；这属于刷新时机导致的正常现象，不代表 `_bootstrap`、写入或搜索接口失败。

##### 最小请求示例

```auto
PUT _data_stream/logs-nginx/_bootstrap
```

##### 请求体参数

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `template_name` | string | `ds-bootstrap-{name}` | 自动创建模板时使用的模板名 |
| `timestamp_field` | string | `@timestamp` | 自动创建模板时使用的时间戳字段名 |
| `priority` | long | `1000` | 自动创建模板的优先级，必须大于等于 `0` |
| `settings.number_of_shards` | integer | 不设置 | 自动创建模板时写入的主分片数，也支持等价写法 `settings.index.number_of_shards` |
| `settings.number_of_replicas` | integer | 不设置 | 自动创建模板时写入的副本数，也支持等价写法 `settings.index.number_of_replicas` |

这些字段仅在本次请求确实需要自动创建默认模板时生效；如果已经存在最高优先级的匹配数据流模板，`_bootstrap` 会直接复用该模板，不会修改已有模板。请求体不支持其他字段。

##### 自动创建默认模板的固定规则

| 项目 | 值 | 说明 |
|------|----|------|
| `index_patterns` | `["{name}"]` | 固定使用数据流名称作为匹配模式 |
| `data_stream.timestamp_field.name` | `@timestamp` 或你传入的 `timestamp_field` | 决定后续写入文档必须携带的时间戳字段 |
| `priority` | `1000` 或你传入的 `priority` | 控制自动创建模板和其他匹配模板的优先级比较 |
| `settings` | 仅支持 `number_of_shards` / `number_of_replicas` 及其 `index.` 前缀等价写法 | 其他 `settings` 字段会报错 |

##### 自定义模板参数

你也可以在请求体里覆盖这些默认值：

```auto
PUT _data_stream/logs-nginx/_bootstrap
{
  "template_name": "logs-nginx-bootstrap",
  "timestamp_field": "event_time",
  "priority": 2001,
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 0
  }
}
```

上面的 `priority: 2001` 是覆盖默认值的示例；如果不传 `priority`，默认值是 `1000`。同理，如果不传 `template_name`，默认会使用 `ds-bootstrap-<data_stream_name>`。

##### 成功响应示例

上面的示例请求如果触发了自动创建模板，响应类似：

```auto
{
  "acknowledged": true,
  "data_stream": "logs-nginx",
  "template_created": true,
  "template_name": "logs-nginx-bootstrap"
}
```

其中 `template_created` 表示本次是否自动创建了模板；只有当它为 `true` 时，响应中才会返回 `template_name`。

##### 接口行为说明

`_bootstrap` 的行为是保守的：

- 如果已经有最高优先级的匹配数据流模板，它只创建数据流，不会覆盖或重建模板
- 如果目标数据流已经存在，再次调用会返回 `400 resource_already_exists_exception`；该接口不是幂等的
- 如果最高优先级的匹配模板是普通组合模板而不是数据流模板，它会直接返回 `400`
- 自动创建出的模板不会在后续失败时回滚

##### 后端或测试的完整验证链路

如果你要按文档完整验证“自动创建模板 + 创建数据流”这条链路，建议使用一个全新的数据流名称，并依次执行：

```auto
PUT _data_stream/logs-nginx-manual/_bootstrap
GET _data_stream/logs-nginx-manual
GET _index_template/ds-bootstrap-logs-nginx-manual
DELETE _data_stream/logs-nginx-manual
DELETE _index_template/ds-bootstrap-logs-nginx-manual
```

上面的验证链路假设本次请求返回 `template_created: true`，并且使用了默认模板名。如果你在请求体里自定义了 `template_name`，请改用响应里的 `template_name`；如果响应里的 `template_created` 为 `false`，说明命中了已有模板，此时可以继续查询数据流和模板，但不要删除并非本次测试创建的已有模板。

##### 常见失败场景

常见失败场景包括：

- `PUT _data_stream/{name}` 不能通过请求体配置数据流；如果缺少匹配模板，请改用 `PUT _data_stream/{name}/_bootstrap`
- `PUT _data_stream/{name}/_bootstrap` 不是幂等接口；如果同名数据流已经存在，会返回 `400 resource_already_exists_exception`
- `priority` 必须大于等于 `0`
- 未知字段会报错
- `settings` 只允许 `number_of_shards` / `number_of_replicas` 及其 `index.` 前缀等价写法
- 当最高优先级的匹配模板是普通组合模板而不是数据流模板时，`_bootstrap` 返回 `400`

### 步骤 3：向数据流中摄入数据

你可以使用常规的索引 API 将数据摄入数据流。请确保每个索引的文档都包含时间戳字段。如果尝试摄入没有时间戳字段的文档，将会报错。

如果你希望在联调时“写完立刻搜到”，建议在写入请求上加 `refresh=wait_for`，或者写完后再主动执行一次 `POST /{data_stream}/_refresh`。

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
        "_index" : ".ds-logs-nginx-000001",
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
| 新增复合 API：补模板并创建数据流 | `PUT` | `/_data_stream/{name}/_bootstrap` |
| 查看数据流 | `GET` | `/_data_stream` 或 `/_data_stream/{name}` |
| 数据流统计 | `GET` | `/_data_stream/_stats` 或 `/_data_stream/{name}/_stats` |
| 删除数据流 | `DELETE` | `/_data_stream/{name}` |
| 滚动数据流 | `POST` | `/{data_stream}/_rollover` |

> 删除操作支持逗号分隔的列表和通配符，如 `DELETE /_data_stream/logs-*`。
> `PUT /_data_stream/{name}` 和“首次写入自动创建”都仍然要求预先存在匹配的数据流模板；只有新增的复合 API `PUT /_data_stream/{name}/_bootstrap` 才会在缺模板时自动补一个默认模板。

---

## 相关文档

- [索引模板]({{< relref "./index-templates" >}})：数据流依赖索引模板定义
- [索引生命周期管理]({{< relref "/docs/features/data-retention/ilm" >}})：自动管理数据流的滚动与删除
- [数据生命周期]({{< relref "/docs/features/data-retention/lifecycle" >}})：完整的数据保留策略体系

