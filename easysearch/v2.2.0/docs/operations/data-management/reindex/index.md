---
title: "重建索引"
date: 0001-01-01
description: "使用 Reindex 进行数据迁移、模型变更和索引重建。"
summary: "重新索引数据 #  相关指南 #    索引管理：创建、删除与重建索引  别名（Aliases）  创建索引后，如果您需要进行广泛的更改，例如为每个文档添加一个新字段或合并多个索引以形成一个新的索引，而不是删除索引，使更改脱机，然后重新索引数据，则可以使用 reindex 操作。
使用 reindex 操作，可以将通过查询选择的所有文档或文档子集复制到另一个索引。重新索引是一个 POST 操作。在最基本的形式中，指定源索引和目标索引。
 重新编制索引可能是一项昂贵的操作，具体取决于源索引的大小。我们建议您通过将 number_of_replicas 设置为 0 来禁用目标索引中的副本，并在重新索引过程完成后重新启用它们。
 重新索引所有文档 #  您可以将所有文档从一个索引复制到另一个索引。
首先需要使用所需的字段映射和设置创建目标索引，或者可以从源索引中复制这些映射和设置：
PUT destination { &#34;mappings&#34;:{ &#34;Add in your desired mappings&#34; }, &#34;settings&#34;:{ &#34;Add in your desired settings&#34; } } reindex 命令将所有文档从源索引复制到目标索引：
POST _reindex { &#34;source&#34;:{ &#34;index&#34;:&#34;source&#34; }, &#34;dest&#34;:{ &#34;index&#34;:&#34;destination&#34; } } 如果尚未创建目标索引，则 reindex 操作将使用默认配置创建新的目标索引。
从远程群集 reindex #  您可以从远程集群中的索引复制文档。使用 remote 选项指定远程主机名和所需的登录凭据。"
---


# 重新索引数据

## 相关指南

- [索引管理：创建、删除与重建索引]({{< relref "/docs/operations/data-management/index-management" >}})
- [别名（Aliases）]({{< relref "/docs/operations/data-management/aliases" >}})

创建索引后，如果您需要进行广泛的更改，例如为每个文档添加一个新字段或合并多个索引以形成一个新的索引，而不是删除索引，使更改脱机，然后重新索引数据，则可以使用 `reindex` 操作。

使用 `reindex` 操作，可以将通过查询选择的所有文档或文档子集复制到另一个索引。重新索引是一个 `POST` 操作。在最基本的形式中，指定源索引和目标索引。

> 重新编制索引可能是一项昂贵的操作，具体取决于源索引的大小。我们建议您通过将 `number_of_replicas` 设置为 `0` 来禁用目标索引中的副本，并在重新索引过程完成后重新启用它们。

## 重新索引所有文档

您可以将所有文档从一个索引复制到另一个索引。

首先需要使用所需的字段映射和设置创建目标索引，或者可以从源索引中复制这些映射和设置：

```json
PUT destination
{
   "mappings":{
      "Add in your desired mappings"
   },
   "settings":{
      "Add in your desired settings"
   }
}
```

`reindex` 命令将所有文档从源索引复制到目标索引：

```json
POST _reindex
{
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination"
   }
}
```

如果尚未创建目标索引，则 `reindex` 操作将使用默认配置创建新的目标索引。

## 从远程群集 reindex

您可以从远程集群中的索引复制文档。使用 `remote` 选项指定远程主机名和所需的登录凭据。

此命令会到达远程集群，使用用户名和密码登录，并将所有文档从该远程集群中的源索引复制到本地集群中的目标索引：

```json
POST _reindex
{
   "source":{
      "remote":{
         "host":"https://<REST_endpoint_of_remote_cluster>:9200",
         "username":"YOUR_USERNAME",
         "password":"YOUR_PASSWORD"
      }
   },
   "dest":{
      "index":"destination"
   }
}
```

您可以指定以下选项：

| 选项              | 有效值    | 描述                                   | 必填 |
| :---------------- | :-------- | :------------------------------------- | :--- |
| `host`            | String    | 远程集群的 REST 端点                   | Yes  |
| `username`        | String    | 登录到远程集群的用户名                 | No   |
| `password`        | String    | 登录到远程群集的密码                   | No   |
| `socket_timeout`  | Time Unit | 套接字读取的等待时间（默认为 30 秒）   | No   |
| `connect_timeout` | Time Unit | 远程连接超时的等待时间（默认为 30 秒） | No   |

## 重新索引文档子集

只能复制与搜索查询匹配的特定文档集。

此命令仅将查询操作匹配的文档子集复制到目标索引：

```json
POST _reindex
{
   "source":{
      "index":"source",
      "query": {
        "match": {
           "field_name": "text"
         }
      }
   },
   "dest":{
      "index":"destination"
   }
}
```

有关所有查询操作的列表，请参见[全文查询]({{< relref "/docs/features/fulltext-search/fulltext-search.md" >}})。

## 合并一个或多个索引

通过将源索引添加为列表，可以组合一个或多个索引中的文档。

此命令将所有文档从两个源索引复制到一个目标索引：

```json
POST _reindex
{
   "source":{
      "index":[
         "source_1",
         "source_2"
      ]
   },
   "dest":{
      "index":"destination"
   }
}
```

确保源索引和目标索引的碎片数量相同。

## 仅重索引缺少的文档

通过将 `op_type` 选项设置为 `create` ，可以仅复制目标索引中缺少的文档。
在这种情况下，如果已经存在具有相同 ID 的文档，则操作将忽略源索引中的文档。
要忽略文档的所有版本冲突，请将 `conflicts` 选项设置为 `proceed` 。

```json
POST _reindex
{
   "conflicts":"proceed",
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination",
      "op_type":"create"
   }
}
```

## 重新索引排序的文档

对文档中的特定字段进行排序后，可以复制某些文档。

此命令基于 `timestamp` 字段复制最后 10 个文档：

```json
POST _reindex
{
   "size":10,
   "source":{
      "index":"source",
      "sort":{
         "timestamp":"desc"
      }
   },
   "dest":{
      "index":"destination"
   }
}
```

## 重新索引期间转换文档

您可以使用 `script` 选项在重新索引过程中转换数据。
我们建议在 Easysearch 中编写脚本时使用 Painless。

此命令通过 Painless 脚本运行源索引，该脚本在将 `account` 对象复制到目标索引之前增加 `number` 字段：

```json
POST _reindex
{
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination"
   },
   "script":{
      "lang":"painless",
      "source":"ctx._account.number++"
   }
}
```

您还可以指定一个摄取管道，以在重新索引过程中转换数据。

首先必须创建一个定义了 `processors` 的管道。您可以在 ingest 管道中使用许多不同的 `processors`。

这是一个示例摄取管道，它定义了一个 `split` 处理器，该处理器基于 `space` 分隔符拆分 `text` 字段，并将其存储在新的 `word` 字段中。 `script` 处理器是一个无痛脚本，它查找 `word` 字段的长度并将其存储在新的 `word_count` 字段中。 `remove` 处理器删除 `test` 字段。

```json
PUT _ingest/pipeline/pipeline-test
{
"description": "Splits the text field into a list. Computes the length of the 'word' field and stores it in a new 'word_count' field. Removes the 'test' field.",
"processors": [
 {
   "split": {
     "field": "text",
     "separator": "\\s+",
     "target_field": "word"
   },
 }
 {
   "script": {
     "lang": "painless",
     "source": "ctx.word_count = ctx.word.length"
   }
 },
 {
   "remove": {
     "field": "test"
   }
 }
]
}
```

创建管道后，可以使用 `reindex` 操作：

```json
POST _reindex
{
  "source": {
    "index": "source",
  },
  "dest": {
    "index": "destination",
    "pipeline": "pipeline-test"
  }
}
```

## 更新当前索引中的文档

要更新当前索引中的数据而不将其复制到其他索引，请使用 `update_by_query` 操作。

`update_by_query` 操作是一次可以对单个索引执行的 `POST` 操作。

```json
POST <index_name>/_update_by_query
```

如果在没有参数的情况下运行此命令，则会增加索引中所有文档的版本号。

---

## 异步执行

对于大量数据的重新索引，建议使用异步模式，避免长时间等待超时：

```json
POST _reindex?wait_for_completion=false
{
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination"
   }
}
```

异步模式立即返回一个 `task_id`，可通过 Task API 查询进度：

```json
GET _tasks/<task_id>
```

响应中包含任务状态、已处理的文档数和进度信息。

---

## 限流控制

使用 `requests_per_second` 参数限制重新索引的速度，避免对集群造成过大压力：

```json
POST _reindex?requests_per_second=500
{
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination"
   }
}
```

设置为 `-1` 表示不限流（默认行为）。

### 动态调整限流（Rethrottle）

在重新索引任务执行过程中，可以动态调整限流速度：

```json
POST _reindex/<task_id>/_rethrottle?requests_per_second=1000
```

设置为 `-1` 可取消限流。

---

## 切片并行

使用 `slices` 参数将重新索引操作切分为多个子任务并行执行，显著提升速度：

```json
POST _reindex?slices=5
{
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination"
   }
}
```

也可以设置为 `auto`，让 Easysearch 自动根据源索引的分片数决定切片数量：

```json
POST _reindex?slices=auto
{
   "source":{
      "index":"source"
   },
   "dest":{
      "index":"destination"
   }
}
```

> **建议**：`slices` 数量不应超过源索引的分片数。`auto` 模式通常是最佳选择。

---

## 查询参数

`_reindex` API 支持以下查询参数：

| 参数 | 类型 | 描述 | 默认值 |
| :--- | :--- | :--- | :----- |
| `refresh` | `boolean` | 完成后是否刷新受影响的索引 | `false` |
| `timeout` | `time` | 每个批量请求等待不可用分片的时间 | `1m` |
| `wait_for_active_shards` | `string` | 操作前需要活跃的分片副本数。设置为 `all` 表示所有副本 | `1` |
| `wait_for_completion` | `boolean` | 是否等待操作完成再返回。`false` 时返回 `task_id` | `true` |
| `requests_per_second` | `number` | 子请求限流速率。`-1` 表示不限流 | `-1` |
| `scroll` | `time` | 搜索上下文的保持时间 | `5m` |
| `slices` | `number/string` | 任务切片数。可设为数字或 `auto` | `1` |
| `max_docs` | `number` | 最大处理文档数 | 全部 |

## 源索引选项

可以为源索引指定以下选项：

| 选项 | 类型 | 描述 | 必填 |
| :--- | :--- | :--- | :--- |
| `index` | `string/array` | 源索引名称，可提供多个作为列表 | 是 |
| `max_docs` | `integer` | 要重新索引的最大文档数 | 否 |
| `query` | `object` | 用于过滤文档的搜索查询 | 否 |
| `size` | `integer` | 每批滚动的文档数 | 否 |
| `slice` | `object` | 手动切片配置（`id` 和 `max`） | 否 |
| `sort` | `list` | 重新索引前的排序字段 | 否 |
| `remote` | `object` | 远程集群连接信息 | 否 |
| `_source` | `array` | 仅包含指定的源字段 | 否 |

## 目标索引选项

可以为目标索引指定以下选项：

| 选项 | 类型 | 描述 | 必填 |
| :--- | :--- | :--- | :--- |
| `index` | `string` | 目标索引的名称 | 是 |
| `version_type` | `enum` | 版本类型：`internal`、`external`、`external_gt`、`external_gte` | 否 |
| `op_type` | `string` | 设置为 `create` 可仅索引不存在的文档 | 否 |
| `pipeline` | `string` | 摄取管道名称，在索引前对文档进行处理 | 否 |
| `routing` | `string` | 路由值（`keep`、`discard` 或自定义值） | 否 |

---

## 相关文档

- [索引管理]({{< relref "/docs/operations/data-management/index-management" >}})：索引 CRUD 操作
- [数据生命周期]({{< relref "/docs/features/data-retention/lifecycle" >}})：完整的数据保留策略

