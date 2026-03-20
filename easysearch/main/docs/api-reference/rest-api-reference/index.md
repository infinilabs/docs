---
title: "常用 API 速查"
date: 0001-01-01
summary: "Easysearch REST API #  async_search.delete #  删除指定的异步搜索请求及其结果。
DELETE _async_search/{id} URL 参数 #     参数 类型 说明     id string 必需。异步搜索请求的 ID。    async_search.get #  获取异步搜索请求的当前状态和可用结果。
GET _async_search/{id} URL 参数 #     参数 类型 说明     id string 必需。异步搜索请求的 ID。   wait_for_completion_timeout string 等待搜索完成的超时时间。   keep_alive string 结果保留的时间长度。    async_search.stats #  获取集群中异步搜索的统计信息。"
---


# Easysearch REST API

## async_search.delete

删除指定的异步搜索请求及其结果。

```
DELETE _async_search/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--- | :----- | :----------------------------------------------------------------------- |
| id   | string | 必需。异步搜索请求的 ID。                                                 |

## async_search.get

获取异步搜索请求的当前状态和可用结果。

```
GET _async_search/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------------- | :----- | :----------------------------------------------------------------------- |
| id                           | string | 必需。异步搜索请求的 ID。                                                 |
| wait_for_completion_timeout  | string | 等待搜索完成的超时时间。                                                   |
| keep_alive                   | string | 结果保留的时间长度。                                                       |

## async_search.stats

获取集群中异步搜索的统计信息。

```
GET _async_search/stats
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :----- | :----------------------------------------------------------------------- |
| nodeId   | string | 以逗号分隔的节点 ID 列表。仅返回指定节点的统计信息。                       |

## async_search.submit

提交一个异步搜索请求。返回一个 ID，可用于后续获取结果。

```
POST _async_search
POST {index}/_async_search
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------------- | :------- | :----------------------------------------------------------------------- |
| index                        | string   | 以逗号分隔的索引名称。                                                   |
| wait_for_completion_timeout  | string   | 等待搜索完成的超时时间。                                                   |
| keep_alive                   | string   | 结果保留的时间长度。                                                       |
| keep_on_completion           | boolean  | 搜索完成后是否保留结果。默认为 `false`。                                   |
| request_cache                | boolean  | 是否使用请求缓存。                                                         |
| batched_reduce_size          | number   | 用于批量归约的分片数。                                                     |

#### HTTP 请求体

标准搜索请求体，包含 query、aggregations 等。

## bulk

在单个请求中执行多个索引、更新和/或删除操作。

```
POST _bulk
PUT _bulk
```

```
POST {index}/_bulk
PUT {index}/_bulk
```

```
POST {index}/{type}/_bulk
PUT {index}/{type}/_bulk
```

#### HTTP 请求体

操作定义和数据（action-data 对），以换行符分隔。

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| wait_for_active_shards | string  | 在执行批量操作前，必须处于活跃状态的分片副本数。默认为 1（仅主分片）。设为 `all` 表示所有副本。也可设为不超过分片总副本数（副本数 + 1）的任意非负值。 |
| refresh                | enum    | 如果为 `true`，刷新受影响的分片使操作对搜索可见。如果为 `wait_for`，等待刷新完成后再返回。如果为 `false`（默认），不执行刷新操作。                                                                                                |
| routing                | string  | 指定路由值。                                                                                                                                                                                                                                                                                            |
| timeout                | time    | 显式操作超时时间。                                                                                                                                                                                                                                                                                        |
| type                   | string  | 未指定类型的文档的默认类型。                                                                                                                                                                                                                                                            |
| \_source               | list    | 是否返回 \_source 字段，或指定默认返回的字段列表，可在每个子请求中覆盖。                                                                                                                                                                                 |
| \_source_excludes      | list    | 从返回的 \_source 字段中排除的默认字段列表，可在每个子请求中覆盖。                                                                                                                                                                                                         |
| \_source_includes      | list    | 从 \_source 字段中提取并返回的默认字段列表，可在每个子请求中覆盖。                                                                                                                                                                                                       |
| pipeline               | string  | 用于预处理文档的 Pipeline ID。                                                                                                                                                                                                                                                             |
| require_alias          | boolean | 为所有写入文档设置 require_alias，默认为 false。                                                                                                                                                                                                                                          |

## cat.aliases

显示当前已配置的索引别名信息，包括过滤器和路由信息。

```
GET _cat/aliases
```

```
GET _cat/aliases/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------- | :------ | :--------------------------------------------------------------------------------------- |
| format           | string  | Accept 头的简写形式，如 json、yaml                                    |
| local            | boolean | 返回本地信息，不从主节点获取状态（默认：false）    |
| h                | list    | 要显示的列名，逗号分隔                                          |
| help             | boolean | 返回帮助信息                                                                  |
| s                | list    | 用于排序的列名或列别名，逗号分隔                        |
| v                | boolean | 详细模式，显示列标题                                                     |
| expand_wildcards | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。 |

## cat.allocation

显示每个数据节点的分片分配数量和磁盘使用情况快照。

```
GET _cat/allocation
```

```
GET _cat/allocation/{node_id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| format         | string  | Accept 头的简写形式，如 json、yaml                                 |
| bytes          | enum    | 显示字节值的单位                                              |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| h              | list    | 要显示的列名，逗号分隔                                       |
| help           | boolean | 返回帮助信息                                                               |
| s              | list    | 用于排序的列名或列别名，逗号分隔                     |
| v              | boolean | 详细模式，显示列标题                                                  |

## cat.count

快速获取整个集群或单个索引的文档数量。

```
GET _cat/count
```

```
GET _cat/count/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------- | :------ | :---------------------------------------------------------------- |
| format    | string  | Accept 头的简写形式，如 json、yaml             |
| h         | list    | 要显示的列名，逗号分隔                   |
| help      | boolean | 返回帮助信息                                           |
| s         | list    | 用于排序的列名或列别名，逗号分隔 |
| v         | boolean | 详细模式，显示列标题                              |

## cat.fielddata

显示集群中每个数据节点上 fielddata 当前使用的堆内存大小。

```
GET _cat/fielddata
```

```
GET _cat/fielddata/{fields}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------- | :------ | :---------------------------------------------------------------- |
| format    | string  | Accept 头的简写形式，如 json、yaml             |
| bytes     | enum    | 显示字节值的单位                          |
| h         | list    | 要显示的列名，逗号分隔                   |
| help      | boolean | 返回帮助信息                                           |
| s         | list    | 用于排序的列名或列别名，逗号分隔 |
| v         | boolean | 详细模式，显示列标题                              |
| fields    | list    | 要在输出中返回的字段列表，逗号分隔          |

## cat.health

返回集群健康状态的简要信息。

```
GET _cat/health
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------- | :------ | :---------------------------------------------------------------- |
| format    | string  | Accept 头的简写形式，如 json、yaml             |
| h         | list    | 要显示的列名，逗号分隔                   |
| help      | boolean | 返回帮助信息                                           |
| s         | list    | 用于排序的列名或列别名，逗号分隔 |
| time      | enum    | 显示时间值的单位                          |
| ts        | boolean | 设为 false 可禁用时间戳                              |
| v         | boolean | 详细模式，显示列标题                              |

## cat.help

返回 Cat API 的帮助信息。

```
GET _cat
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------- | :------ | :---------------------------------------------------------------- |
| help      | boolean | 返回帮助信息                                           |
| s         | list    | 用于排序的列名或列别名，逗号分隔 |

## cat.indices

返回索引相关信息：主分片和副本数量、文档数量、磁盘大小等。

```
GET _cat/indices
```

```
GET _cat/indices/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------------ | :------ | :------------------------------------------------------------------------------------------------------- |
| format                    | string  | Accept 头的简写形式，如 json、yaml                                                    |
| bytes                     | enum    | 显示字节值的单位                                                                 |
| local                     | boolean | 返回本地信息，不从主节点获取状态（默认：false）                    |
| master_timeout            | time    | 连接主节点的显式超时时间                                                 |
| h                         | list    | 要显示的列名，逗号分隔                                                          |
| health                    | enum    | 按健康状态过滤索引（"green"、"yellow" 或 "red"） |
| help                      | boolean | 返回帮助信息                                                                                  |
| pri                       | boolean | 设为 true 仅返回主分片的统计信息                                                      |
| s                         | list    | 用于排序的列名或列别名，逗号分隔                                        |
| time                      | enum    | 显示时间值的单位                                                                 |
| v                         | boolean | 详细模式，显示列标题                                                                     |
| include_unloaded_segments | boolean | 如果设为 true，段统计将包含未加载到内存中的段   |
| expand_wildcards          | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                 |

## cat.master

返回主节点的信息。

```
GET _cat/master
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| format         | string  | Accept 头的简写形式，如 json、yaml                                 |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| h              | list    | 要显示的列名，逗号分隔                                       |
| help           | boolean | 返回帮助信息                                                               |
| s              | list    | 用于排序的列名或列别名，逗号分隔                     |
| v              | boolean | 详细模式，显示列标题                                                  |

## cat.nodeattrs

返回自定义节点属性信息。

```
GET _cat/nodeattrs
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| format         | string  | Accept 头的简写形式，如 json、yaml                                 |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| h              | list    | 要显示的列名，逗号分隔                                       |
| help           | boolean | 返回帮助信息                                                               |
| s              | list    | 用于排序的列名或列别名，逗号分隔                     |
| v              | boolean | 详细模式，显示列标题                                                  |

## cat.nodes

返回集群节点的基本性能统计信息。

```
GET _cat/nodes
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :----------------------------------------------------------------------------------------------------------------- |
| bytes          | enum    | 显示字节值的单位                                                                           |
| format         | string  | Accept 头的简写形式，如 json、yaml                                                              |
| full_id        | boolean | 返回完整的节点 ID 而非缩写版本（默认：false） |
| local          | boolean | 使用本地集群状态而非主节点状态来计算所选节点（默认：false） |
| master_timeout | time    | 连接主节点的显式超时时间                                                           |
| h              | list    | 要显示的列名，逗号分隔                                                                    |
| help           | boolean | 返回帮助信息                                                                                            |
| s              | list    | 用于排序的列名或列别名，逗号分隔                                                  |
| time           | enum    | 显示时间值的单位                                                                           |
| v              | boolean | 详细模式，显示列标题                                                                               |

## cat.pending_tasks

返回集群待处理任务的简要信息。

```
GET _cat/pending_tasks
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| format         | string  | Accept 头的简写形式，如 json、yaml                                 |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| h              | list    | 要显示的列名，逗号分隔                                       |
| help           | boolean | 返回帮助信息                                                               |
| s              | list    | 用于排序的列名或列别名，逗号分隔                     |
| time           | enum    | 显示时间值的单位                                              |
| v              | boolean | 详细模式，显示列标题                                                  |

## cat.plugins

返回各节点已安装插件的信息。

```
GET _cat/plugins
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| format         | string  | Accept 头的简写形式，如 json、yaml                                 |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| h              | list    | 要显示的列名，逗号分隔                                       |
| help           | boolean | 返回帮助信息                                                               |
| s              | list    | 用于排序的列名或列别名，逗号分隔                     |
| v              | boolean | 详细模式，显示列标题                                                  |

## cat.recovery

返回正在进行和已完成的分片恢复信息。

```
GET _cat/recovery
```

```
GET _cat/recovery/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------- | :------ | :------------------------------------------------------------------------------------------- |
| format      | string  | Accept 头的简写形式，如 json、yaml                                        |
| active_only | boolean | 如果为 `true`，响应仅包含正在进行的分片恢复 |
| bytes       | enum    | 显示字节值的单位                                                     |
| detailed    | boolean | 如果为 `true`，响应包含分片恢复的详细信息 |
| h           | list    | 要显示的列名，逗号分隔                                              |
| help        | boolean | 返回帮助信息                                                                      |
| index       | list    | 逗号分隔的索引名称列表或通配符表达式，用于限制返回的信息 |
| s           | list    | 用于排序的列名或列别名，逗号分隔                            |
| time        | enum    | 显示时间值的单位                                                     |
| v           | boolean | 详细模式，显示列标题                                                         |

## cat.repositories

返回集群中已注册的快照仓库信息。

```
GET _cat/repositories
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------- |
| format         | string  | Accept 头的简写形式，如 json、yaml                |
| local          | boolean | 返回本地信息，不从主节点获取状态 |
| master_timeout | time    | 连接主节点的显式超时时间             |
| h              | list    | 要显示的列名，逗号分隔                      |
| help           | boolean | 返回帮助信息                                              |
| s              | list    | 用于排序的列名或列别名，逗号分隔    |
| v              | boolean | 详细模式，显示列标题                                 |

## cat.segments

提供索引分片中段（segment）的底层信息。

```
GET _cat/segments
```

```
GET _cat/segments/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------- | :------ | :---------------------------------------------------------------- |
| format    | string  | Accept 头的简写形式，如 json、yaml             |
| bytes     | enum    | 显示字节值的单位                          |
| h         | list    | 要显示的列名，逗号分隔                   |
| help      | boolean | 返回帮助信息                                           |
| s         | list    | 用于排序的列名或列别名，逗号分隔 |
| v         | boolean | 详细模式，显示列标题                              |

## cat.shards

返回节点上分片分配的详细视图。

```
GET _cat/shards
```

```
GET _cat/shards/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| format         | string  | Accept 头的简写形式，如 json、yaml                                 |
| bytes          | enum    | 显示字节值的单位                                              |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| h              | list    | 要显示的列名，逗号分隔                                       |
| help           | boolean | 返回帮助信息                                                               |
| s              | list    | 用于排序的列名或列别名，逗号分隔                     |
| time           | enum    | 显示时间值的单位                                              |
| v              | boolean | 详细模式，显示列标题                                                  |

## cat.snapshots

返回特定仓库中的所有快照。

```
GET _cat/snapshots
```

```
GET _cat/snapshots/{repository}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :---------------------------------------------------------------- |
| format             | string  | Accept 头的简写形式，如 json、yaml             |
| ignore_unavailable | boolean | 设为 true 以忽略不可用的快照 |
| master_timeout     | time    | 连接主节点的显式超时时间          |
| h                  | list    | 要显示的列名，逗号分隔                   |
| help               | boolean | 返回帮助信息                                           |
| s                  | list    | 用于排序的列名或列别名，逗号分隔 |
| time               | enum    | 显示时间值的单位                          |
| v                  | boolean | 详细模式，显示列标题                              |

## cat.tasks

返回集群中一个或多个节点上当前正在执行的任务信息。

```
GET _cat/tasks
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| format         | string  | Accept 头的简写形式，如 json、yaml                                                                                                                                               |
| nodes          | list    | 用于限制返回信息的节点 ID 或名称列表（逗号分隔）；使用 `_local` 返回当前连接节点的信息，留空获取所有节点的信息 |
| actions        | list    | 要返回的操作列表（逗号分隔）。留空返回全部。                                                                                                               |
| detailed       | boolean | 返回详细的任务信息（默认：false） |
| parent_task_id | string  | 返回指定父任务 ID 的任务（node_id:task_number）。设为 -1 返回全部。                                                                                                          |
| h              | list    | 要显示的列名，逗号分隔                                                                                                                                                     |
| help           | boolean | 返回帮助信息                                                                                                                                                                             |
| s              | list    | 用于排序的列名或列别名，逗号分隔                                                                                                                                   |
| time           | enum    | 显示时间值的单位                                                                                                                                                            |
| v              | boolean | 详细模式，显示列标题                                                                                                                                                                |

## cat.templates

返回已存在的索引模板信息。

```
GET _cat/templates
```

```
GET _cat/templates/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| format         | string  | Accept 头的简写形式，如 json、yaml                                 |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| h              | list    | 要显示的列名，逗号分隔                                       |
| help           | boolean | 返回帮助信息                                                               |
| s              | list    | 用于排序的列名或列别名，逗号分隔                     |
| v              | boolean | 详细模式，显示列标题                                                  |

## cat.thread_pool

返回各节点的集群范围线程池统计信息。
默认返回所有线程池的活跃（active）、队列（queue）和拒绝（rejected）统计信息。

```
GET _cat/thread_pool
```

```
GET _cat/thread_pool/{thread_pool_patterns}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| format         | string  | Accept 头的简写形式，如 json、yaml                                 |
| size           | enum    | 显示值的倍率                                             |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| h              | list    | 要显示的列名，逗号分隔                                       |
| help           | boolean | 返回帮助信息                                                               |
| s              | list    | 用于排序的列名或列别名，逗号分隔                     |
| v              | boolean | 详细模式，显示列标题                                                  |

## clear_scroll

显式清除搜索滚动上下文，释放相关资源。

```
DELETE _search/scroll
```

```
DELETE _search/scroll/{scroll_id}
```

#### HTTP 请求体

如未通过 scroll_id 参数指定，则为要清除的滚动 ID 的逗号分隔列表

## cluster.allocation_explain

解释集群中分片分配的原因。

```
GET _cluster/allocation/explain
POST _cluster/allocation/explain
```

#### HTTP 请求体

需要解释的索引、分片和主分片标识。为空表示'解释第一个未分配的分片'

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------------------- | :------ | :------------------------------------------------------------------- |
| include_yes_decisions | boolean | 在解释中返回 'YES' 决策（默认：false） |
| include_disk_info     | boolean | 返回磁盘使用和分片大小信息（默认：false） |

## cluster.delete_component_template

删除组件模板。

```
DELETE _component_template/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| timeout        | time | 显式操作超时时间 |
| master_timeout | time | 指定连接主节点的超时时间 |

## cluster.delete_voting_config_exclusions

清除集群投票配置排除项。

```
DELETE _cluster/voting_config_exclusions
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------- | :------ | :---------------------------------------------------------------------------------------------------------------------------------------- |
| wait_for_removal | boolean | 指定是否等待所有被排除的节点从集群中移除后再清除投票配置排除列表。 |

## cluster.exists_component_template

检查指定组件模板是否存在。

```
HEAD _component_template/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |

## cluster.get_component_template

获取一个或多个组件模板。

```
GET _component_template
```

```
GET _component_template/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |

## cluster.get_settings

返回集群级别的设置。

```
GET _cluster/settings
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------- | :------ | :------------------------------------------------------- |
| flat_settings    | boolean | 以扁平格式返回设置（默认：false）          |
| master_timeout   | time    | 连接主节点的显式超时时间 |
| timeout          | time    | 显式操作超时时间 |
| include_defaults | boolean | 是否返回所有默认集群设置。 |

## cluster.health

返回集群健康状态的基本信息。

```
GET _cluster/health
```

```
GET _cluster/health/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------------------ | :------ | :--------------------------------------------------------------------------------------- |
| expand_wildcards                | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。 |
| level                           | enum    | 指定返回信息的详细级别 |
| local                           | boolean | 返回本地信息，不从主节点获取状态（默认：false）    |
| master_timeout                  | time    | 连接主节点的显式超时时间                                 |
| timeout                         | time    | 显式操作超时时间 |
| wait_for_active_shards          | string  | 等待直到指定数量的分片处于活跃状态 |
| wait_for_nodes                  | string  | 等待直到指定数量的节点可用 |
| wait_for_events                 | enum    | 等待直到所有给定优先级的当前排队事件都已处理 |
| wait_for_no_relocating_shards   | boolean | 是否等待直到集群中没有正在迁移的分片 |
| wait_for_no_initializing_shards | boolean | 是否等待直到集群中没有正在初始化的分片 |
| wait_for_status                 | enum    | 等待直到集群处于特定状态 |

## cluster.pending_tasks

返回所有尚未执行的集群级变更（例如创建索引、更新映射、
分配或失败分片）的列表。

```
GET _cluster/pending_tasks
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |
| master_timeout | time    | 指定连接主节点的超时时间                                              |

## cluster.post_voting_config_exclusions

通过节点 ID 或名称更新集群投票配置排除列表。

```
POST _cluster/voting_config_exclusions
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :----- | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| node_ids   | string | 要从投票配置中排除的节点持久化 ID 的逗号分隔列表。如果指定了此参数，则不能同时指定 ?node_names。 |
| node_names | string | 要从投票配置中排除的节点名称的逗号分隔列表。如果指定了此参数，则不能同时指定 ?node_ids。 |
| timeout    | time   | 显式操作超时时间 |

## cluster.put_component_template

创建或更新组件模板。

```
PUT _component_template/{name}
POST _component_template/{name}
```

#### HTTP 请求体

模板定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :----------------------------------------------------------------------------------------- |
| create         | boolean | 索引模板是否仅在为新模板时添加，还是也可以替换现有模板 |
| timeout        | time    | 显式操作超时时间 |
| master_timeout | time    | 指定连接主节点的超时时间                                                   |

## cluster.put_settings

更新集群级别的设置。

```
PUT _cluster/settings
```

#### HTTP 请求体

要更新的设置。可以是 `transient`（临时）或 `persistent`（持久化，重启后保留）。

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------- |
| flat_settings  | boolean | 以扁平格式返回设置（默认：false）          |
| master_timeout | time    | 连接主节点的显式超时时间 |
| timeout        | time    | 显式操作超时时间 |

## cluster.remote_info

返回所有已配置的远程集群信息。

```
GET _remote/info
```

## cluster.reroute

允许手动更改集群中各分片的分配。

```
POST _cluster/reroute
```

#### HTTP 请求体

要执行的 `commands` 定义（`move`、`cancel`、`allocate`）

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------------- |
| dry_run        | boolean | 仅模拟操作并返回结果状态 |
| explain        | boolean | 返回命令可以或不能执行的原因说明 |
| retry_failed   | boolean | 重试因过多连续分配失败而被阻止的分片分配 |
| metric         | list    | 将返回的信息限制在指定的指标。默认返回除元数据外的全部信息        |
| master_timeout | time    | 连接主节点的显式超时时间                                     |
| timeout        | time    | 显式操作超时时间 |

## cluster.state

返回集群状态的详细信息。

```
GET _cluster/state
```

```
GET _cluster/state/{metric}
```

```
GET _cluster/state/{metric}/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------------ | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| local                     | boolean | 返回本地信息，不从主节点获取状态（默认：false）                                                                      |
| master_timeout            | time    | 指定连接主节点的超时时间                                                                                                                   |
| flat_settings             | boolean | 以扁平格式返回设置（默认：false）                                                                                                            |
| wait_for_metadata_version | number  | 等待元数据版本等于或大于指定的元数据版本                                                                   |
| wait_for_timeout          | time    | 等待 wait_for_metadata_version 的最大超时时间                                                                                   |
| ignore_unavailable        | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices          | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards          | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |

## cluster.stats

返回集群的高级统计概览。

```
GET _cluster/stats
```

```
GET _cluster/stats/nodes/{node_id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------ | :------ | :---------------------------------------------- |
| flat_settings | boolean | 以扁平格式返回设置（默认：false） |
| timeout       | time    | 显式操作超时时间 |

## count

返回匹配查询的文档数量。

```
POST _count
GET _count
```

```
POST {index}/_count
GET {index}/_count
```

```
POST {index}/{type}/_count
GET {index}/{type}/_count
```

#### HTTP 请求体

使用 Query DSL 指定的查询条件，用于限制结果（可选）

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| ignore_throttled   | boolean | 当被节流时，是否忽略指定的具体、扩展或别名索引                                                                   |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| min_score          | number  | 仅在结果中包含具有特定 `_score` 值的文档 |
| preference         | string  | 指定操作应在哪个节点或分片上执行（默认：随机）                                                                           |
| routing            | list    | 指定路由值列表，逗号分隔                                                                                                          |
| q                  | string  | Lucene 查询字符串语法的查询                                                                                                                    |
| analyzer           | string  | 用于查询字符串的分析器                                                                                                                   |
| analyze_wildcard   | boolean | 是否分析通配符和前缀查询（默认：false）                                                                            |
| default_operator   | enum    | 查询字符串查询的默认运算符（AND 或 OR）                                                                                                    |
| df                 | string  | 查询字符串中未指定字段前缀时使用的默认字段 |
| lenient            | boolean | 是否忽略基于格式的查询失败（如向数字字段提供文本）                                                  |
| terminate_after    | number  | 每个分片的最大计数，达到该值时查询将提前终止 |

## create

在索引中创建新文档。

当索引中已存在相同 ID 的文档时返回 409 错误。

```
PUT {index}/_create/{id}
POST {index}/_create/{id}
```

```
PUT {index}/{type}/{id}/_create
POST {index}/{type}/{id}/_create
```

#### HTTP 请求体

The document

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :----- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| wait_for_active_shards | string | 在执行索引操作前，必须处于活跃状态的分片副本数。默认为 1（仅主分片）。设为 `all` 表示所有副本，也可设为不超过分片总副本数的任意非负值。 |
| refresh                | enum   | 如果为 `true`，刷新受影响的分片使操作对搜索可见；如果为 `wait_for`，等待刷新完成后再返回；如果为 `false`（默认），不执行刷新操作。                                                                                     |
| routing                | string | 特定的路由值 |
| timeout                | time   | 显式操作超时时间 |
| version                | number | 用于并发控制的显式版本号                                                                                                                                                                                                                                                                   |
| version_type           | enum   | 指定版本类型                                                                                                                                                                                                                                                                                             |
| pipeline               | string | 用于预处理传入文档的管道 ID |

## create_pit

创建 Point-In-Time（PIT）上下文，为后续搜索请求提供一致的数据快照视图。

```
POST {index}/_pit
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------------- | :------ | :----------------------------------------------------------------------- |
| keep_alive                   | time    | PIT 上下文的保持时间（必需） |
| allow_partial_pit_creation   | boolean | 是否允许在部分分片不可用时仍创建 PIT（默认：true） |
| preference                   | string  | 指定执行搜索的分片偏好 |
| routing                      | string  | 特定的路由值 |
| expand_wildcards             | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引 |
| ignore_unavailable           | boolean | 当指定的具体索引不可用时是否忽略 |
| allow_no_indices             | boolean | 当通配符表达式不匹配任何具体索引时是否忽略 |

## dangling_indices.delete_dangling_index

删除指定的悬空索引

```
DELETE _dangling/{index_uuid}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------- | :------ | :-------------------------------------------------------- |
| accept_data_loss | boolean | 必须设为 true 才能删除悬空索引 |
| timeout          | time    | 显式操作超时时间 |
| master_timeout   | time    | 指定连接主节点的超时时间                  |

## dangling_indices.import_dangling_index

导入指定的悬空索引

```
POST _dangling/{index_uuid}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------- | :------ | :-------------------------------------------------------- |
| accept_data_loss | boolean | 必须设为 true 才能导入悬空索引 |
| timeout          | time    | 显式操作超时时间 |
| master_timeout   | time    | 指定连接主节点的超时时间                  |

## dangling_indices.list_dangling_indices

返回所有悬空索引。

```
GET _dangling
```

## delete

从索引中删除文档。

```
DELETE {index}/_doc/{id}
```

```
DELETE {index}/{type}/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :----- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| wait_for_active_shards | string | 设置在执行删除操作之前必须处于活跃状态的分片副本数。默认为 1，即仅主分片。设为 `all` 表示所有分片副本，否则设为小于或等于分片总副本数（副本数 + 1）的任意非负值 |
| refresh                | enum   | 如果为 `true`，刷新受影响的分片使操作对搜索可见；如果为 `wait_for`，等待刷新完成后再返回；如果为 `false`（默认），不执行刷新操作。                                                                                      |
| routing                | string | 特定的路由值 |
| timeout                | time   | 显式操作超时时间 |
| if_seq_no              | number | 仅当最后修改文档的操作具有指定的序列号时才执行删除                                                                                                                                                                                            |
| if_primary_term        | number | 仅当最后修改文档的操作具有指定的主任期时才执行删除                                                                                                                                                                                               |
| version                | number | 用于并发控制的显式版本号                                                                                                                                                                                                                                                                    |
| version_type           | enum   | 指定版本类型                                                                                                                                                                                                                                                                                              |

## delete_by_query

删除匹配查询的文档。

```
POST {index}/_delete_by_query
```

```
POST {index}/{type}/_delete_by_query
```

#### HTTP 请求体

使用 Query DSL 的搜索定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| analyzer               | string  | 用于查询字符串的分析器                                                                                                                                                                                                                                                                                    |
| analyze_wildcard       | boolean | 是否分析通配符和前缀查询（默认：false）                                                                                                                                                                                                                                             |
| default_operator       | enum    | 查询字符串查询的默认运算符（AND 或 OR）                                                                                                                                                                                                                                                                     |
| df                     | string  | 查询字符串中未指定字段前缀时使用的默认字段 |
| from                   | number  | 起始偏移量（默认：0） |
| ignore_unavailable     | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                                                                                                                                                                                   |
| allow_no_indices       | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。                                                                                                                                                                  |
| conflicts              | enum    | 当 delete-by-query 遇到版本冲突时的行为                                                                                                                                                                                                                                                                 |
| expand_wildcards       | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                                                                                                                                                                                    |
| lenient                | boolean | 是否忽略基于格式的查询失败（如向数字字段提供文本）                                                                                                                                                                                                                   |
| preference             | string  | 指定操作应在哪个节点或分片上执行（默认：随机）                                                                                                                                                                                                                                            |
| q                      | string  | Lucene 查询字符串语法的查询                                                                                                                                                                                                                                                                                     |
| routing                | list    | 指定路由值列表，逗号分隔                                                                                                                                                                                                                                                                           |
| scroll                 | time    | 指定滚动搜索的索引一致性视图应维持多长时间                                                                                                                                                                                                                                    |
| search_type            | enum    | 搜索操作类型                                                                                                                                                                                                                                                                                                       |
| search_timeout         | time    | 每个搜索请求的显式超时时间。默认无超时。 |
| size                   | number  | 已弃用，请改用 `max_docs` |
| max_docs               | number  | 要处理的最大文档数（默认：所有文档） |
| sort                   | list    | 逗号分隔的 <field>:<direction> 对列表 |
| \_source               | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes      | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes      | list    | 要从 \_source 字段中提取并返回的字段列表 |
| terminate_after        | number  | 每个分片收集的最大文档数，达到此数量后查询将提前终止。                                                                                                                                                                                                    |
| stats                  | list    | 请求的特定 'tag'，用于日志和统计目的 |
| version                | boolean | 是否在命中结果中返回文档版本号                                                                                                                                                                                                                                                                 |
| request_cache          | boolean | 指定此请求是否使用请求缓存，默认使用索引级别设置 |
| refresh                | boolean | 受影响的索引是否应该刷新？                                                                                                                                                                                                                                                                                   |
| timeout                | time    | 每个批量请求等待不可用分片的时间。                                                                                                                                                                                                                                              |
| wait_for_active_shards | string  | 设置在执行按查询删除操作之前必须处于活跃状态的分片副本数。默认为 1，即仅主分片。设为 `all` 表示所有分片副本，否则设为小于或等于分片总副本数（副本数 + 1）的任意非负值 |
| scroll_size            | number  | 驱动 delete-by-query 的滚动请求大小                                                                                                                                                                                                                                                                     |
| wait_for_completion    | boolean | 请求是否应阻塞直到按查询删除操作完成。 |
| requests_per_second    | number  | 此请求的节流值（每秒子请求数）。-1 表示不节流。                                                                                                                                                                                                                                             |
| slices                 | number\|string | 该任务应被分成的切片数。默认为 1（不拆分子任务），可设为 `auto`。 |

## delete_by_query_rethrottle

修改 delete-by-query 操作的每秒请求数。

```
POST _delete_by_query/{task_id}/_rethrottle
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------ | :----- | :------------------------------------------------------------------------------------------------- |
| requests_per_second | number | 此请求的节流值（每秒浮点子请求数）。-1 表示不节流。 |

## delete_pit

删除一个或多个 Point-In-Time（PIT）上下文。支持删除指定的 PIT（通过请求体）或一次性删除所有 PIT。

```
DELETE _pit
```

```
DELETE _pit/_all
```

#### HTTP 请求体

包含要删除的 PIT ID 列表的 JSON 对象。使用 `_pit/_all` 路由时无需请求体。

## delete_script

删除存储的脚本。

```
DELETE _scripts/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| timeout        | time | 显式操作超时时间 |
| master_timeout | time | 指定连接主节点的超时时间 |

## exists

检查指定 ID 的文档是否存在于索引中。

```
HEAD {index}/_doc/{id}
```

```
HEAD {index}/{type}/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------- | :------ | :------------------------------------------------------------------------------- |
| stored_fields     | list    | 逗号分隔的要在响应中返回的存储字段列表 |
| preference        | string  | 指定操作应在哪个节点或分片上执行（默认：随机） |
| realtime          | boolean | 指定是否以实时模式或搜索模式执行操作 |
| refresh           | boolean | 在执行操作之前刷新包含该文档的分片 |
| routing           | string  | 特定的路由值 |
| \_source          | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes | list    | 要从 \_source 字段中提取并返回的字段列表 |
| version           | number  | 用于并发控制的显式版本号                                  |
| version_type      | enum    | 指定版本类型                                                            |

## exists_source

检查指定 ID 文档的 _source 是否存在于索引中。

```
HEAD {index}/_source/{id}
```

```
HEAD {index}/{type}/{id}/_source
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------- | :------ | :------------------------------------------------------------------------------- |
| preference        | string  | 指定操作应在哪个节点或分片上执行（默认：随机） |
| realtime          | boolean | 指定是否以实时模式或搜索模式执行操作 |
| refresh           | boolean | 在执行操作之前刷新包含该文档的分片 |
| routing           | string  | 特定的路由值 |
| \_source          | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes | list    | 要从 \_source 字段中提取并返回的字段列表 |
| version           | number  | 用于并发控制的显式版本号                                  |
| version_type      | enum    | 指定版本类型                                                            |

## explain

解释某个文档为什么匹配（或不匹配）某个查询。

```
GET {index}/_explain/{id}
POST {index}/_explain/{id}
```

```
GET {index}/{type}/{id}/_explain
POST {index}/{type}/{id}/_explain
```

#### HTTP 请求体

使用 Query DSL 的查询定义

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------- | :------ | :--------------------------------------------------------------------------------------------------------- |
| analyze_wildcard  | boolean | 指定查询字符串中的通配符和前缀查询是否应被分析（默认：false） |
| analyzer          | string  | 查询字符串查询使用的分析器 |
| default_operator  | enum    | 查询字符串查询的默认运算符（AND 或 OR）                                                    |
| df                | string  | 查询字符串查询的默认字段（默认：\_all） |
| stored_fields     | list    | 逗号分隔的要在响应中返回的存储字段列表 |
| lenient           | boolean | 是否忽略基于格式的查询失败（如向数字字段提供文本）  |
| preference        | string  | 指定操作应在哪个节点或分片上执行（默认：随机）                           |
| q                 | string  | Lucene 查询字符串语法的查询                                                                    |
| routing           | string  | 特定的路由值 |
| \_source          | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes | list    | 要从 \_source 字段中提取并返回的字段列表 |

## field_caps

返回多个索引中各字段的能力信息（可搜索、可聚合等）。

```
GET _field_caps
POST _field_caps
```

```
GET {index}/_field_caps
POST {index}/_field_caps
```

#### HTTP 请求体

使用 Query DSL 指定的索引过滤器

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| fields             | list    | 逗号分隔的字段名称列表 |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| include_unmapped   | boolean | 指示是否应在响应中包含未映射的字段。 |

## get

获取指定文档。

```
GET {index}/_doc/{id}
```

```
GET {index}/{type}/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------- | :------ | :------------------------------------------------------------------------------- |
| stored_fields     | list    | 逗号分隔的要在响应中返回的存储字段列表 |
| preference        | string  | 指定操作应在哪个节点或分片上执行（默认：随机） |
| realtime          | boolean | 指定是否以实时模式或搜索模式执行操作 |
| refresh           | boolean | 在执行操作之前刷新包含该文档的分片 |
| routing           | string  | 特定的路由值 |
| \_source          | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes | list    | 要从 \_source 字段中提取并返回的字段列表 |
| version           | number  | 用于并发控制的显式版本号                                  |
| version_type      | enum    | 指定版本类型                                                            |

## get_all_pits

获取集群中所有节点上的所有活跃 Point-In-Time（PIT）上下文。

```
GET _pit/_all
```

此 API 无需额外参数。响应包含一个 `pits` 数组，列出所有活跃的 PIT 及其详细信息。

## get_script

获取存储的脚本。

```
GET _scripts/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| master_timeout | time | 指定连接主节点的超时时间 |

## get_script_context

返回所有可用的脚本上下文。

```
GET _script_context
```

## get_script_languages

返回可用的脚本类型、语言和上下文。

```
GET _script_language
```

## get_source

获取指定文档的 _source 内容。

```
GET {index}/_source/{id}
```

```
GET {index}/{type}/{id}/_source
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------- | :------ | :------------------------------------------------------------------------------- |
| preference        | string  | 指定操作应在哪个节点或分片上执行（默认：随机） |
| realtime          | boolean | 指定是否以实时模式或搜索模式执行操作 |
| refresh           | boolean | 在执行操作之前刷新包含该文档的分片 |
| routing           | string  | 特定的路由值 |
| \_source          | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes | list    | 要从 \_source 字段中提取并返回的字段列表 |
| version           | number  | 用于并发控制的显式版本号                                  |
| version_type      | enum    | 指定版本类型                                                            |

## index

在索引中创建或更新文档。

```
PUT {index}/_doc/{id}
POST {index}/_doc/{id}
```

```
POST {index}/_doc
```

```
POST {index}/{type}
```

```
PUT {index}/{type}/{id}
POST {index}/{type}/{id}
```

#### HTTP 请求体

The document

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| wait_for_active_shards | string  | 在执行索引操作前，必须处于活跃状态的分片副本数。默认为 1（仅主分片）。设为 `all` 表示所有副本，也可设为不超过分片总副本数的任意非负值。 |
| op_type                | enum    | 显式操作类型。对于有显式文档 ID 的请求默认为 `index`，对于没有显式文档 ID 的请求默认为 `create` |
| refresh                | enum    | 如果为 `true`，刷新受影响的分片使操作对搜索可见；如果为 `wait_for`，等待刷新完成后再返回；如果为 `false`（默认），不执行刷新操作。                                                                                     |
| routing                | string  | 特定的路由值 |
| timeout                | time    | 显式操作超时时间 |
| version                | number  | 用于并发控制的显式版本号                                                                                                                                                                                                                                                                   |
| version_type           | enum    | 指定版本类型                                                                                                                                                                                                                                                                                             |
| if_seq_no              | number  | 仅当最后修改文档的操作具有指定的序列号时才执行索引操作                                                                                                                                                                                            |
| if_primary_term        | number  | 仅当最后修改文档的操作具有指定的主任期时才执行索引操作                                                                                                                                                                                               |
| pipeline               | string  | 用于预处理传入文档的管道 ID |
| require_alias          | boolean | 当为 true 时，要求目标必须是别名。默认为 false |

## ik.reload

在集群的所有节点上重新加载 IK 分词器词典。可以指定从特定索引加载词典数据。

```
POST _ik/_reload
```

#### HTTP 请求体

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| dict_key   | string   | 要重新加载的词典键名。                                                     |
| dict_index | string   | 词典数据所在的索引名称。默认为 `.analysis_ik`。                            |

## ilm.add_policy

为一个或多个索引添加 ISM（索引状态管理）策略。

```
POST _ilm/add
POST _ilm/add/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| index    | string   | 以逗号分隔的索引名称。                                                   |

#### HTTP 请求体

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| policy_id  | string   | 要添加的 ISM 策略 ID。                                                   |

## ilm.change_policy

更改已应用于一个或多个索引的 ISM 策略。

```
POST _ilm/change_policy
POST _ilm/change_policy/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| index    | string   | 以逗号分隔的索引名称。                                                   |

#### HTTP 请求体

变更策略配置，包含新的 `policy_id`、状态过滤器等。

## ilm.delete_policy

删除指定的 ISM 策略。

```
DELETE _ilm/policy/{policyID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| policyID   | string   | 必需。要删除的策略 ID。                                                   |
| refresh    | string   | 控制请求所做更改何时可被搜索到。有效值：`true`、`false`、`wait_for`。       |

## ilm.explain

获取一个或多个索引的 ISM 策略执行状态和详情。

```
GET _ilm/explain
GET _ilm/explain/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------- | :----------------------------------------------------------------------- |
| index                  | string   | 以逗号分隔的索引名称。                                                   |
| local                  | boolean  | 是否仅从本地节点返回信息。默认为 `false`。                                 |
| show_policy            | boolean  | 是否在结果中包含策略定义。                                                 |
| show_validate_action   | boolean  | 是否显示验证动作信息。                                                     |
| size                   | number   | 每页返回数量（用于列表模式）。                                             |
| from                   | number   | 分页起始位置。                                                             |
| sortField              | string   | 排序字段。                                                                 |
| sortOrder              | string   | 排序方向。有效值：`asc`、`desc`。                                          |
| queryString            | string   | 过滤查询字符串。                                                           |

## ilm.get_policy

获取一个或多个 ISM 策略的定义。

```
GET _ilm/policy
GET _ilm/policy/{policyID}
HEAD _ilm/policy/{policyID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| policyID       | string   | 策略 ID。省略则列出所有策略。                                              |
| size           | number   | 每页返回数量（列表模式）。                                                 |
| from           | number   | 分页起始位置（列表模式）。                                                 |
| sortField      | string   | 排序字段。                                                                 |
| sortOrder      | string   | 排序方向。有效值：`asc`、`desc`。                                          |
| queryString    | string   | 过滤查询字符串。                                                           |

## ilm.put_policy

创建或更新一个 ISM 策略。

```
PUT _ilm/policy/{policyID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| policyID       | string   | 必需。策略 ID。                                                           |
| if_seq_no      | number   | 仅在序列号匹配时执行操作，用于乐观并发控制。                                 |
| if_primary_term| number   | 仅在主要任期匹配时执行操作，用于乐观并发控制。                               |
| refresh        | string   | 控制请求所做更改何时可被搜索到。有效值：`true`、`false`、`wait_for`。       |

#### HTTP 请求体

ISM 策略定义，包含策略描述、状态列表、转换条件、操作配置等。

## ilm.remove_policy

从一个或多个索引上移除 ISM 策略。

```
POST _ilm/remove
POST _ilm/remove/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| index    | string   | 必需。以逗号分隔的索引名称。                                               |

## ilm.retry

重试因错误而失败的 ISM 策略执行。

```
POST _ilm/retry
POST _ilm/retry/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| index    | string   | 必需。以逗号分隔的索引名称。                                               |

#### HTTP 请求体

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| state    | string   | 要重试的状态名称。                                                         |

## indices.add_block

为索引添加操作限制块。

```
PUT {index}/_block/{block}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| timeout            | time    | 显式操作超时时间 |
| master_timeout     | time    | 指定连接主节点的超时时间                                                                                                                   |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |

## indices.analyze

对文本执行分析（分词）处理，返回分词结果。

```
GET _analyze
POST _analyze
```

```
GET {index}/_analyze
POST {index}/_analyze
```

#### HTTP 请求体

定义分析器/分词器参数以及需要进行分析的文本

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------- | :----- | :------------------------------------------- |
| index     | string | 操作范围限定的索引名称 |

## indices.clear_cache

清除一个或多个索引的全部或指定类型的缓存。

```
POST _cache/clear
```

```
POST {index}/_cache/clear
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| fielddata          | boolean | 清除字段数据                                                                                                                                           |
| fields             | list    | 使用 `fielddata` 参数时要清除的字段的逗号分隔列表（默认：全部） |
| query              | boolean | 清除查询缓存 |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| index              | list    | 用于限制操作的逗号分隔的索引名称列表 |
| request            | boolean | 清除请求缓存                                                                                                                                        |

## indices.clone

克隆索引

```
PUT {index}/_clone/{target}
POST {index}/_clone/{target}
```

#### HTTP 请求体

目标索引的配置（`settings` 和 `aliases`）

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :----- | :-------------------------------------------------------------------------------------------- |
| timeout                | time   | 显式操作超时时间 |
| master_timeout         | time   | 指定连接主节点的超时时间                                                      |
| wait_for_active_shards | string | 设置操作返回前等待克隆索引上活跃分片的数量。 |

## indices.close

关闭索引。

```
POST {index}/_close
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| timeout                | time    | 显式操作超时时间 |
| master_timeout         | time    | 指定连接主节点的超时时间                                                                                                                   |
| ignore_unavailable     | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices       | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards       | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| wait_for_active_shards | string  | 设置操作返回前等待的活跃分片数。 |

## indices.create

创建索引，可指定设置和映射。

```
PUT {index}
```

#### HTTP 请求体

索引的配置（`settings` 和 `mappings`）

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :------------------------------------------------------------------------ |
| include_type_name      | boolean | 映射体中是否应包含类型。 |
| wait_for_active_shards | string  | 设置操作返回前等待的活跃分片数量。 |
| timeout                | time    | 显式操作超时时间 |
| master_timeout         | time    | 指定连接主节点的超时时间                                  |

## indices.create_data_stream

创建或更新数据流。

```
PUT _data_stream/{name}
```

#### HTTP 请求体

数据流定义。

---

## indices.data_streams_stats

返回数据流的操作统计信息。

```
GET _data_stream/_stats
```

```
GET _data_stream/{name}/_stats
```

---

## indices.delete_data_stream

删除数据流。

```
DELETE _data_stream/{name}
```

---

## indices.get_data_stream

获取数据流信息。

```
GET _data_stream
```

```
GET _data_stream/{name}
```

---

## indices.delete

删除索引。

```
DELETE {index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :----------------------------------------------------------------------------------------- |
| timeout            | time    | 显式操作超时时间 |
| master_timeout     | time    | 指定连接主节点的超时时间                                                   |
| ignore_unavailable | boolean | 忽略不可用的索引（默认：false） |
| allow_no_indices   | boolean | 如果通配符表达式未匹配到任何具体索引则忽略（默认：false） |
| expand_wildcards   | enum    | 通配符表达式是否应扩展到打开或关闭的索引（默认：open） |

## indices.delete_alias

删除索引别名。

```
DELETE {index}/_alias/{name}
```

```
DELETE {index}/_aliases/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| timeout        | time | 文档的显式时间戳      |
| master_timeout | time | 指定连接主节点的超时时间 |

## indices.delete_index_template

删除索引模板。

```
DELETE _index_template/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| timeout        | time | 显式操作超时时间 |
| master_timeout | time | 指定连接主节点的超时时间 |

## indices.delete_template

删除索引模板。

```
DELETE _template/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| timeout        | time | 显式操作超时时间 |
| master_timeout | time | 指定连接主节点的超时时间 |

## indices.disk_usage

分析一个或多个索引的磁盘空间使用情况，按字段级别提供详细的空间占用报告。此操作资源开销较大，需显式设置 `run_expensive_tasks=true`。

```
POST {index}/_disk_usage
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------- | :------ | :----------------------------------------------------------------------- |
| run_expensive_tasks  | boolean | 必须设为 `true` 才能执行此操作，否则请求将被拒绝（安全保护措施） |
| flush                | boolean | 分析前是否先刷新索引（默认：true） |
| expand_wildcards     | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引 |
| ignore_unavailable   | boolean | 当指定的具体索引不可用时是否忽略 |
| allow_no_indices     | boolean | 当通配符表达式不匹配任何具体索引时是否忽略 |

## indices.exists

检查指定索引是否存在。

```
HEAD {index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :----------------------------------------------------------------------------------------- |
| local              | boolean | 返回本地信息，不从主节点获取状态（默认：false）      |
| ignore_unavailable | boolean | 忽略不可用的索引（默认：false） |
| allow_no_indices   | boolean | 如果通配符表达式未匹配到任何具体索引则忽略（默认：false） |
| expand_wildcards   | enum    | 通配符表达式是否应扩展到打开或关闭的索引（默认：open） |
| flat_settings      | boolean | 以扁平格式返回设置（默认：false）                                            |
| include_defaults   | boolean | 是否返回每个索引的所有默认设置。                             |

## indices.exists_alias

检查指定索引别名是否存在。

```
HEAD _alias/{name}
```

```
HEAD {index}/_alias/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| local              | boolean | 返回本地信息，不从主节点获取状态（默认：false）                                                                      |

## indices.exists_index_template

检查指定索引模板是否存在。

```
HEAD _index_template/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| flat_settings  | boolean | 以扁平格式返回设置（默认：false）                                       |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |

## indices.exists_template

检查指定索引模板是否存在。

```
HEAD _template/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| flat_settings  | boolean | 以扁平格式返回设置（默认：false）                                       |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |

## indices.exists_type

检查指定文档类型是否存在（已废弃）。

```
HEAD {index}/_mapping/{type}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| local              | boolean | 返回本地信息，不从主节点获取状态（默认：false）                                                                      |

## indices.field_usage_stats

返回一个或多个索引的字段使用统计信息，显示各字段被查询和使用的情况。

该接口的统计采集受索引级动态配置 `index.field_usage_stats.enabled` 控制；当该配置为 `false` 时，不再记录字段访问统计，并返回空统计结果。

```
GET {index}/_field_usage_stats
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :----------------------------------------------------------------------- |
| fields             | list    | 逗号分隔的字段列表，指定要报告使用情况的字段（默认：所有字段） |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引 |
| ignore_unavailable | boolean | 当指定的具体索引不可用时是否忽略 |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略 |

## indices.flush

对一个或多个索引执行 flush 操作，将内存中的数据持久化到磁盘。

```
POST _flush
GET _flush
```

```
POST {index}/_flush
GET {index}/_flush
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| force              | boolean | 是否强制执行 flush，即使没有需要提交的变更。这在需要递增事务日志 ID 但没有未提交变更时很有用（此设置可视为内部设置）。 |
| wait_if_ongoing    | boolean | 如果设为 true，当另一个 flush 操作正在执行时，将阻塞等待。默认为 true。如果设为 false，当另一个 flush 已在运行时将跳过。                             |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                                                                                                                                    |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。                                                                                                                   |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                                                                                                                                     |

## indices.flush_synced

对一个或多个索引执行同步 flush 操作。同步 flush 已废弃，请使用 flush 代替。

```
POST _flush/synced
GET _flush/synced
```

```
POST {index}/_flush/synced
GET {index}/_flush/synced
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |

## indices.forcemerge

对一个或多个索引执行强制合并操作。

```
POST _forcemerge
```

```
POST {index}/_forcemerge
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| flush                | boolean | 是否在操作后刷新索引（默认：true）                                                                 |
| ignore_unavailable   | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices     | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards     | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| max_num_segments     | number  | 索引应合并成的段数（默认：动态）                                                                                  |
| only_expunge_deletes | boolean | 指定操作是否仅清除已删除的文档 |

## indices.get

返回一个或多个索引的信息。

```
GET {index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :----------------------------------------------------------------------------------------- |
| include_type_name  | boolean | 是否在响应中添加类型名称（默认：false） |
| local              | boolean | 返回本地信息，不从主节点获取状态（默认：false）      |
| ignore_unavailable | boolean | 忽略不可用的索引（默认：false） |
| allow_no_indices   | boolean | 如果通配符表达式未匹配到任何具体索引则忽略（默认：false） |
| expand_wildcards   | enum    | 通配符表达式是否应扩展到打开或关闭的索引（默认：open） |
| flat_settings      | boolean | 以扁平格式返回设置（默认：false）                                            |
| include_defaults   | boolean | 是否返回每个索引的所有默认设置。                             |
| master_timeout     | time    | 指定连接主节点的超时时间                                                   |

## indices.get_alias

获取索引别名信息。

```
GET _alias
```

```
GET _alias/{name}
```

```
GET {index}/_alias/{name}
```

```
GET {index}/_alias
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| local              | boolean | 返回本地信息，不从主节点获取状态（默认：false）                                                                      |

## indices.get_field_mapping

获取一个或多个字段的映射信息。

```
GET _mapping/field/{fields}
```

```
GET {index}/_mapping/field/{fields}
```

```
GET _mapping/{type}/field/{fields}
```

```
GET {index}/_mapping/{type}/field/{fields}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| include_type_name  | boolean | 映射体中是否应返回类型。 |
| include_defaults   | boolean | 是否也应返回默认映射值 |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| local              | boolean | 返回本地信息，不从主节点获取状态（默认：false）                                                                      |

## indices.get_index_template

获取索引模板。

```
GET _index_template
```

```
GET _index_template/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| flat_settings  | boolean | 以扁平格式返回设置（默认：false）                                       |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |

## indices.get_mapping

获取一个或多个索引的映射信息。

```
GET _mapping
```

```
GET {index}/_mapping
```

```
GET _mapping/{type}
```

```
GET {index}/_mapping/{type}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| include_type_name  | boolean | 是否在响应中添加类型名称（默认：false） |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| master_timeout     | time    | 指定连接主节点的超时时间                                                                                                                   |
| local              | boolean | 返回本地信息，不从主节点获取状态（默认：false）                                                                      |

## indices.get_settings

获取一个或多个索引的设置信息。

```
GET _settings
```

```
GET {index}/_settings
```

```
GET {index}/_settings/{name}
```

```
GET _settings/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| master_timeout     | time    | 指定连接主节点的超时时间                                                                                                                   |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| flat_settings      | boolean | 以扁平格式返回设置（默认：false）                                                                                                            |
| local              | boolean | 返回本地信息，不从主节点获取状态（默认：false）                                                                      |
| include_defaults   | boolean | 是否返回每个索引的所有默认设置。                                                                                             |

## indices.get_template

获取索引模板。

```
GET _template
```

```
GET _template/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------- | :------ | :------------------------------------------------------------------------------------ |
| include_type_name | boolean | 映射体中是否应返回类型。 |
| flat_settings     | boolean | 以扁平格式返回设置（默认：false）                                       |
| master_timeout    | time    | 连接主节点的显式超时时间                              |
| local             | boolean | 返回本地信息，不从主节点获取状态（默认：false） |

## indices.get_upgrade

\_upgrade API 已不再有用，将被移除。

```
GET _upgrade
```

```
GET {index}/_upgrade
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |

## indices.open

打开已关闭的索引。

```
POST {index}/_open
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| timeout                | time    | 显式操作超时时间 |
| master_timeout         | time    | 指定连接主节点的超时时间                                                                                                                   |
| ignore_unavailable     | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices       | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards       | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| wait_for_active_shards | string  | 设置操作返回前等待的活跃分片数。 |

## indices.put_alias

创建或更新索引别名。

```
PUT {index}/_alias/{name}
POST {index}/_alias/{name}
```

```
PUT {index}/_aliases/{name}
POST {index}/_aliases/{name}
```

#### HTTP 请求体

别名的设置，例如 `routing` 或 `filter`

**必需**：否

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| timeout        | time | 文档的显式时间戳      |
| master_timeout | time | 指定连接主节点的超时时间 |

## indices.put_index_template

创建或更新索引模板。

```
PUT _index_template/{name}
POST _index_template/{name}
```

#### HTTP 请求体

模板定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :----------------------------------------------------------------------------------------- |
| create         | boolean | 索引模板是否仅在为新模板时添加，还是也可以替换现有模板 |
| cause          | string  | 用户定义的创建/更新索引模板原因 |
| master_timeout | time    | 指定连接主节点的超时时间                                                   |

## indices.put_mapping

更新索引的映射。

```
PUT {index}/_mapping
POST {index}/_mapping
```

```
PUT {index}/{type}/_mapping
POST {index}/{type}/_mapping
```

```
PUT {index}/_mapping/{type}
POST {index}/_mapping/{type}
```

```
PUT {index}/{type}/_mappings
POST {index}/{type}/_mappings
```

```
PUT {index}/_mappings/{type}
POST {index}/_mappings/{type}
```

```
PUT _mappings/{type}
POST _mappings/{type}
```

```
PUT {index}/_mappings
POST {index}/_mappings
```

```
PUT _mapping/{type}
POST _mapping/{type}
```

#### HTTP 请求体

映射定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| include_type_name  | boolean | 映射体中是否应包含类型。 |
| timeout            | time    | 显式操作超时时间 |
| master_timeout     | time    | 指定连接主节点的超时时间                                                                                                                   |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| write_index_only   | boolean | 当为 true 时，仅将映射应用于别名或数据流的写入索引 |

## indices.put_settings

更新索引的设置。

```
PUT _settings
```

```
PUT {index}/_settings
```

#### HTTP 请求体

要更新的索引设置

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------- |
| master_timeout    | time    | 指定连接主节点的超时时间                                                                                     |
| timeout           | time    | 显式操作超时时间 |
| preserve_existing  | boolean | 是否更新已有设置。如果设为 `true`，索引上的已有设置保持不变，默认为 `false`。 |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略 |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况） |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引 |
| flat_settings      | boolean | 以扁平格式返回设置（默认：false） |

## indices.put_template

创建或更新索引模板。

```
PUT _template/{name}
POST _template/{name}
```

#### HTTP 请求体

模板定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------ |
| include_type_name | boolean | 映射体中是否应返回类型。 |
| order             | number  | 合并多个匹配模板时此模板的顺序（数字越大越后合并，会覆盖较小数字的设置） |
| create            | boolean | 索引模板是否仅在为新模板时添加，还是也可以替换现有模板 |
| master_timeout    | time    | 指定连接主节点的超时时间                                                                                        |

## indices.recovery

返回正在进行的分片恢复信息。

```
GET _recovery
```

```
GET {index}/_recovery
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------- | :------ | :----------------------------------------------------------- |
| detailed    | boolean | 是否显示分片恢复的详细信息 |
| active_only | boolean | 仅显示当前正在进行的恢复 |

## indices.refresh

对一个或多个索引执行刷新操作，使最近的变更对搜索可见。

```
POST _refresh
GET _refresh
```

```
POST {index}/_refresh
GET {index}/_refresh
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |

## indices.refresh_search_analyzers

刷新一个或多个索引的搜索分析器，使分析器配置更改生效，无需关闭和重新打开索引。

```
POST _refresh_search_analyzers
POST _refresh_search_analyzers/{index}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| index    | string   | 必需。以逗号分隔的索引名称。                                               |

## indices.reload

重新加载一个或多个索引的 fast-terms 数据。

```
GET {index}/_reload
POST {index}/_reload
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| index    | string   | 必需。以逗号分隔的索引名称。                                               |

## indices.resolve_index

返回所有匹配的索引、别名和数据流的信息

```
GET _resolve/index/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------- | :--- | :----------------------------------------------------------------------------------------- |
| expand_wildcards | enum | 通配符表达式是否应扩展到打开或关闭的索引（默认：open） |

## indices.rollover

当现有索引被认为太大或太旧时，更新别名以指向新索引。

```
POST {alias}/_rollover
```

```
POST {alias}/_rollover/{new_index}
```

#### HTTP 请求体

执行滚动更新（rollover）需要满足的条件

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------------- |
| include_type_name      | boolean | 映射体中是否应包含类型。 |
| timeout                | time    | 显式操作超时时间 |
| dry_run                | boolean | 如果设为 true，滚动更新操作将仅进行验证而不实际执行，即使条件匹配也是如此。默认为 false |
| master_timeout         | time    | 指定连接主节点的超时时间                                                                                               |
| wait_for_active_shards | string  | 设置操作返回前在新创建的滚动更新索引上等待的活跃分片数。 |

## indices.segments

返回 Lucene 索引中段的底层信息。

```
GET _segments
```

```
GET {index}/_segments
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| verbose            | boolean | 包含 Lucene 的详细内存使用情况。 |

## indices.shard_stores

返回索引分片副本的存储信息。

```
GET _shard_stores
```

```
GET {index}/_shard_stores
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| status             | list    | 用于过滤分片以获取存储信息的逗号分隔的状态列表 |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |

## indices.shrink

将现有索引收缩为具有更少主分片的新索引。

```
PUT {index}/_shrink/{target}
POST {index}/_shrink/{target}
```

#### HTTP 请求体

目标索引的配置（`settings` 和 `aliases`）

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :---------------------------------------------------------------------------------------------- |
| copy_settings          | boolean | 是否从源索引复制设置（默认：false） |
| timeout                | time    | 显式操作超时时间 |
| master_timeout         | time    | 指定连接主节点的超时时间                                                        |
| wait_for_active_shards | string  | 设置操作返回前等待收缩索引上活跃分片的数量。 |

## indices.simulate_index_template

模拟给定索引名称与系统中索引模板的匹配结果。

```
POST _index_template/_simulate_index/{name}
```

#### HTTP 请求体

新的索引模板定义，将包含在模拟中，如同它已存在于系统中

**必需**：否

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :----------------------------------------------------------------------------------------------------------------------------------- |
| create         | boolean | 可选地在请求体中定义的索引模板是否仅在为新模板时进行模拟添加，还是也可以替换现有模板 |
| cause          | string  | 用户定义的模拟创建新模板的原因 |
| master_timeout | time    | 指定连接主节点的超时时间                                                                                             |

## indices.simulate_template

模拟解析给定的模板名称或模板体。

```
POST _index_template/_simulate
```

```
POST _index_template/_simulate/{name}
```

#### HTTP 请求体

如未指定索引模板名称，则为要模拟的新索引模板定义

**必需**：否

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :----------------------------------------------------------------------------------------------------------------------------------- |
| create         | boolean | 可选地在请求体中定义的索引模板是否仅在为新模板时进行模拟添加，还是也可以替换现有模板 |
| cause          | string  | 用户定义的模拟创建新模板的原因 |
| master_timeout | time    | 指定连接主节点的超时时间                                                                                             |

## indices.split

将现有索引拆分为具有更多主分片的新索引。

```
PUT {index}/_split/{target}
POST {index}/_split/{target}
```

#### HTTP 请求体

目标索引的配置（`settings` 和 `aliases`）

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :---------------------------------------------------------------------------------------------- |
| copy_settings          | boolean | 是否从源索引复制设置（默认：false） |
| timeout                | time    | 显式操作超时时间 |
| master_timeout         | time    | 指定连接主节点的超时时间                                                        |
| wait_for_active_shards | string  | 设置操作返回前等待收缩索引上活跃分片的数量。 |

## indices.stats

返回索引上发生的操作统计信息。

```
GET _stats
```

```
GET _stats/{metric}
```

```
GET {index}/_stats
```

```
GET {index}/_stats/{metric}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------------- |
| completion_fields          | list    | 用于 `fielddata` 和 `suggest` 索引指标的字段列表（支持通配符），逗号分隔                                       |
| fielddata_fields           | list    | 用于 `fielddata` 索引指标的逗号分隔字段列表（支持通配符） |
| fields                     | list    | 用于 `fielddata` 和 `completion` 索引指标的字段列表（支持通配符），逗号分隔                                    |
| groups                     | list    | 用于 `search` 索引指标的逗号分隔搜索组列表 |
| level                      | enum    | 返回按集群、索引或分片级别聚合的统计信息 |
| types                      | list    | 用于 `indexing` 索引指标的文档类型列表，逗号分隔                                                               |
| include_segment_file_sizes | boolean | 是否报告每个 Lucene 索引文件的聚合磁盘使用情况（仅在请求段统计时适用） |
| include_unloaded_segments  | boolean | 如果设为 true，段统计将包含未加载到内存中的段                                 |
| expand_wildcards           | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                               |
| forbid_closed_indices      | boolean | 如果设为 false，当显式指定或 expand_wildcards 扩展到关闭的索引时，也会从关闭的索引收集统计信息 |

## indices.update_aliases

批量更新索引别名。

```
POST _aliases
```

#### HTTP 请求体

要执行的 `actions` 定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| timeout        | time | 请求超时时间 |
| master_timeout | time | 指定连接主节点的超时时间 |

## indices.upgrade

\_upgrade API 已不再有用，将被移除。

```
POST _upgrade
```

```
POST {index}/_upgrade
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| allow_no_indices      | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards      | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| ignore_unavailable    | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| wait_for_completion   | boolean | 指定请求是否应阻塞直到所有段升级完成（默认：false） |
| only_ancient_segments | boolean | 如果为 true，仅升级过时（较旧的 Lucene 主版本）的段 |

## indices.validate_query

验证查询 DSL 的正确性，而不实际执行查询。

```
GET _validate/query
POST _validate/query
```

```
GET {index}/_validate/query
POST {index}/_validate/query
```

```
GET {index}/{type}/_validate/query
POST {index}/{type}/_validate/query
```

#### HTTP 请求体

使用 Query DSL 指定的查询定义

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| explain            | boolean | 返回错误的详细信息 |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| q                  | string  | Lucene 查询字符串语法的查询                                                                                                                    |
| analyzer           | string  | 用于查询字符串的分析器                                                                                                                   |
| analyze_wildcard   | boolean | 是否分析通配符和前缀查询（默认：false）                                                                            |
| default_operator   | enum    | 查询字符串查询的默认运算符（AND 或 OR）                                                                                                    |
| df                 | string  | 查询字符串中未指定字段前缀时使用的默认字段 |
| lenient            | boolean | 是否忽略基于格式的查询失败（如向数字字段提供文本）                                                  |
| rewrite            | boolean | 提供更详细的解释，显示将执行的实际 Lucene 查询。 |
| all_shards         | boolean | 在所有分片上执行验证，而不是每个索引一个随机分片 |

## info

返回集群的基本信息。

```
GET /
```

## ingest.delete_pipeline

删除 Pipeline。

```
DELETE _ingest/pipeline/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :------------------------------------------------------- |
| master_timeout | time | 连接主节点的显式超时时间 |
| timeout        | time | 显式操作超时时间 |

## ingest.get_pipeline

获取 Pipeline 信息。

```
GET _ingest/pipeline
```

```
GET _ingest/pipeline/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :------------------------------------------------------- |
| master_timeout | time | 连接主节点的显式超时时间 |

## ingest.processor_grok

返回内置 Grok 模式列表。

```
GET _ingest/processor/grok
```

## ingest.put_pipeline

创建或更新 Pipeline。

```
PUT _ingest/pipeline/{id}
```

#### HTTP 请求体

摄取管道定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :------------------------------------------------------- |
| master_timeout | time | 连接主节点的显式超时时间 |
| timeout        | time | 显式操作超时时间 |

## ingest.simulate

使用示例文档模拟 Pipeline 的执行效果。

```
GET _ingest/pipeline/_simulate
POST _ingest/pipeline/_simulate
```

```
GET _ingest/pipeline/{id}/_simulate
POST _ingest/pipeline/{id}/_simulate
```

#### HTTP 请求体

模拟定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------- | :------ | :------------------------------------------------------------------------ |
| verbose   | boolean | 详细模式。显示已执行管道中每个处理器的数据输出 |

## license

管理 Easysearch 许可证。支持查看当前许可证信息、应用新许可证和删除许可证。

```
GET _license/info
```

```
POST _license/apply
```

```
DELETE _license
```

#### HTTP 请求体（仅 POST）

包含 `license` 字段的 JSON 对象，值为许可证字符串。

**必需**：是（仅 POST）

#### URL 参数（仅 POST 和 DELETE）

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| master_timeout | time | 指定连接主节点的超时时间 |
| timeout        | time | 显式操作超时时间 |

## match_rules.compile

编译指定规则仓库中的规则。编译后的规则用于 Ingest Pipeline 写入阶段的规则匹配（如 `check_match_rules` 处理器）。

```
POST _match_rules/{repo_id}/_compile
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| repo_id    | string   | 必需。规则仓库 ID。                                                       |

#### HTTP 请求体

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| fields   | array    | 编译时声明的字段列表（用于字段限定与范围匹配等）。                         |
| composite | array    | 联合索引定义（用于多字段组合匹配）。                                       |
| quiet    | boolean  | 是否启用静默模式。                                                         |

说明：请求体是必需的，至少传空对象 `{}`。

## match_rules.delete

删除指定的规则仓库。

```
DELETE _match_rules/{repo_id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| repo_id    | string   | 必需。要删除的规则仓库 ID。                                               |

## match_rules.import

批量导入规则到规则仓库。

```
POST _match_rules/_import
PUT _match_rules/_import
POST _match_rules/{repo_id}/_import
PUT _match_rules/{repo_id}/_import
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| repo_id    | string   | 规则仓库 ID。省略时自动生成 `repo_{timestamp}`。                          |

#### HTTP 请求体

规则导入数据，包含规则定义列表。

语义说明：
- `POST /_match_rules/{repo_id}/_import`：覆盖导入（upsert）
- `PUT /_match_rules/{repo_id}/_import`：追加导入（append）
- `POST /_match_rules/_import`：自动生成 `repo_id` 后覆盖导入
- `PUT /_match_rules/_import`：自动生成 `repo_id` 后按追加语义导入（通常表现为新建规则库）

## mget

在单个请求中获取多个文档。

```
GET _mget
POST _mget
```

```
GET {index}/_mget
POST {index}/_mget
```

```
GET {index}/{type}/_mget
POST {index}/{type}/_mget
```

#### HTTP 请求体

文档标识符；可以是 `docs`（包含完整文档信息）或 `ids`（当 URL 中提供了索引和类型时）。

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------- | :------ | :------------------------------------------------------------------------------- |
| stored_fields     | list    | 逗号分隔的要在响应中返回的存储字段列表 |
| preference        | string  | 指定操作应在哪个节点或分片上执行（默认：随机） |
| realtime          | boolean | 指定是否以实时模式或搜索模式执行操作 |
| refresh           | boolean | 在执行操作之前刷新包含该文档的分片 |
| routing           | string  | 特定的路由值 |
| \_source          | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes | list    | 要从 \_source 字段中提取并返回的字段列表 |

## msearch

在单个请求中执行多个搜索操作。

```
GET _msearch
POST _msearch
```

```
GET {index}/_msearch
POST {index}/_msearch
```

```
GET {index}/{type}/_msearch
POST {index}/{type}/_msearch
```

#### HTTP 请求体

请求定义（元数据-搜索请求定义对），以换行符分隔

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| search_type                   | enum    | 搜索操作类型                                                                                                                                                                                                                                                                                                                                                                                                            |
| max_concurrent_searches       | number  | 控制多搜索 API 同时执行的最大并发搜索数 |
| typed_keys                    | boolean | 指定响应中聚合和建议器名称是否应以其各自类型作为前缀 |
| pre_filter_shard_size         | number  | 预过滤阈值。当搜索请求扩展到的分片数超过此阈值时，会强制执行基于查询重写的预过滤来筛选搜索分片。例如，如果分片根据其重写方法无法匹配任何文档（如日期过滤器为必需但分片范围与查询不相交），此轮预过滤可显著减少分片数量。 |
| max_concurrent_shard_requests | number  | 每个子搜索在每个节点上并发执行的并发分片请求数。此值用于限制搜索对集群的影响，以限制并发分片请求的数量 |
| rest_total_hits_as_int        | boolean | 指示 hits.total 在搜索响应中是渲染为整数还是对象                                                                                                                                                                                                                                                                                                                           |
| ccs_minimize_roundtrips       | boolean | 指示是否应在跨集群搜索请求执行过程中最小化网络往返 |

## msearch_template

在单个请求中使用模板执行多个搜索操作。

```
GET _msearch/template
POST _msearch/template
```

```
GET {index}/_msearch/template
POST {index}/_msearch/template
```

```
GET {index}/{type}/_msearch/template
POST {index}/{type}/_msearch/template
```

#### HTTP 请求体

请求定义（元数据-搜索请求定义对），以换行符分隔

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------------- | :------ | :----------------------------------------------------------------------------------------------------------- |
| search_type             | enum    | 搜索操作类型                                                                                        |
| typed_keys              | boolean | 指定响应中聚合和建议器名称是否应以其各自类型作为前缀 |
| max_concurrent_searches | number  | 控制多搜索 API 同时执行的最大并发搜索数 |
| rest_total_hits_as_int  | boolean | 指示 hits.total 在搜索响应中是渲染为整数还是对象       |
| ccs_minimize_roundtrips | boolean | 指示是否应在跨集群搜索请求执行过程中最小化网络往返 |

## mtermvectors

在单个请求中获取多个文档的词项向量。

```
GET _mtermvectors
POST _mtermvectors
```

```
GET {index}/_mtermvectors
POST {index}/_mtermvectors
```

```
GET {index}/{type}/_mtermvectors
POST {index}/{type}/_mtermvectors
```

#### HTTP 请求体

在此定义 ID、文档、参数或每个文档的参数列表。至少需要提供文档 ID 列表。详见文档。

**必需**：否

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ids              | list    | 逗号分隔的文档 ID 列表。必须将 ids 定义为参数，或在请求体中设置 "ids" 或 "docs" |
| term_statistics  | boolean | 指定是否返回词条总频率和文档频率。适用于所有返回的文档，除非在请求体 "params" 或 "docs" 中另有指定。 |
| field_statistics | boolean | 指定是否返回文档计数、文档频率总和以及词条总频率总和。适用于所有返回的文档，除非在请求体 "params" 或 "docs" 中另有指定。 |
| fields           | list    | 逗号分隔的要返回的字段列表。适用于所有返回的文档，除非在请求体 "params" 或 "docs" 中另有指定。 |
| offsets          | boolean | 指定是否返回词条偏移量。适用于所有返回的文档，除非在请求体 "params" 或 "docs" 中另有指定。 |
| positions        | boolean | 指定是否返回词条位置。适用于所有返回的文档，除非在请求体 "params" 或 "docs" 中另有指定。 |
| payloads         | boolean | 指定是否返回词条负载。适用于所有返回的文档，除非在请求体 "params" 或 "docs" 中另有指定。 |
| preference       | string  | 指定操作应在哪个节点或分片上执行（默认：随机） . Applies to all returned documents unless otherwise specified in body "params" or "docs".                             |
| routing          | string  | 指定路由值。 Applies to all returned documents unless otherwise specified in body "params" or "docs".                                                                                        |
| realtime         | boolean | 指定请求是否为实时而非近实时（默认：true）。 |
| version          | number  | 用于并发控制的显式版本号                                                                                                                                                         |
| version_type     | enum    | 指定版本类型                                                                                                                                                                                   |

## nodes.hot_threads

返回集群中各节点的热线程信息。

```
GET _nodes/hot_threads
```

```
GET _nodes/{node_id}/hot_threads
```

```
GET _cluster/nodes/hotthreads
```

```
GET _cluster/nodes/{node_id}/hotthreads
```

```
GET _nodes/hotthreads
```

```
GET _nodes/{node_id}/hotthreads
```

```
GET _cluster/nodes/hot_threads
```

```
GET _cluster/nodes/{node_id}/hot_threads
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------ | :------ | :--------------------------------------------------------------------------------------------------------------------------------------- |
| interval            | time    | 线程第二次采样的时间间隔 |
| snapshots           | number  | 线程堆栈跟踪的采样次数（默认：10） |
| threads             | number  | 指定要提供信息的线程数（默认：3） |
| ignore_idle_threads | boolean | 不显示处于已知空闲状态的线程，例如等待 socket select 或从空任务队列中拉取（默认：true） |
| type                | enum    | 采样类型（默认：cpu） |
| timeout             | time    | 显式操作超时时间 |

## nodes.info

返回集群中节点的信息。

```
GET _nodes
```

```
GET _nodes/{node_id}
```

```
GET _nodes/{metric}
```

```
GET _nodes/{node_id}/{metric}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------ | :------ | :---------------------------------------------- |
| flat_settings | boolean | 以扁平格式返回设置（默认：false） |
| timeout       | time    | 显式操作超时时间 |

## nodes.reload_secure_settings

重新加载安全设置（keystore）。

```
POST _nodes/reload_secure_settings
```

```
POST _nodes/{node_id}/reload_secure_settings
```

#### HTTP 请求体

包含 Easysearch 密钥库密码的对象

**必需**：否

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------- | :--- | :------------------------- |
| timeout   | time | 显式操作超时时间 |

## nodes.stats

返回集群节点的统计信息。

```
GET _nodes/stats
```

```
GET _nodes/{node_id}/stats
```

```
GET _nodes/stats/{metric}
```

```
GET _nodes/{node_id}/stats/{metric}
```

```
GET _nodes/stats/{metric}/{index_metric}
```

```
GET _nodes/{node_id}/stats/{metric}/{index_metric}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------ |
| completion_fields          | list    | 用于 `fielddata` 和 `suggest` 索引指标的字段列表（支持通配符），逗号分隔                                |
| fielddata_fields           | list    | 用于 `fielddata` 索引指标的逗号分隔字段列表（支持通配符） |
| fields                     | list    | 用于 `fielddata` 和 `completion` 索引指标的字段列表（支持通配符），逗号分隔                             |
| groups                     | boolean | 用于 `search` 索引指标的逗号分隔搜索组列表 |
| level                      | enum    | 返回按索引、节点或分片级别聚合的索引统计信息 |
| types                      | list    | 用于 `indexing` 索引指标的文档类型列表，逗号分隔                                                        |
| timeout                    | time    | 显式操作超时时间 |
| include_segment_file_sizes | boolean | 是否报告每个 Lucene 索引文件的聚合磁盘使用情况（仅在请求段统计时适用） |

## nodes.usage

返回节点上 REST 操作使用情况的低级信息。

```
GET _nodes/usage
```

```
GET _nodes/{node_id}/usage
```

```
GET _nodes/usage/{metric}
```

```
GET _nodes/{node_id}/usage/{metric}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :-------- | :--- | :------------------------- |
| timeout   | time | 显式操作超时时间 |

## ping

检查集群是否正在运行。

```
HEAD
```

## put_script

创建或更新存储的脚本。

```
PUT _scripts/{id}
POST _scripts/{id}
```

```
PUT _scripts/{id}/{context}
POST _scripts/{id}/{context}
```

#### HTTP 请求体

The document

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :----- | :--------------------------------------- |
| timeout        | time   | 显式操作超时时间 |
| master_timeout | time   | 指定连接主节点的超时时间 |
| context        | string | 用于编译脚本的上下文名称 |

## rank_eval

允许对一组典型搜索查询评估排名搜索结果的质量

```
GET _rank_eval
POST _rank_eval
```

```
GET {index}/_rank_eval
POST {index}/_rank_eval
```

#### HTTP 请求体

排名评估搜索定义，包括搜索请求、文档评分和排名指标定义。

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| search_type        | enum    | 搜索操作类型                                                                                                                                      |

## reindex

允许将文档从一个索引复制到另一个索引，可选地过滤源文档
（通过查询过滤）、更改目标索引设置，或从远程集群
获取文档。

```
POST _reindex
```

#### HTTP 请求体

使用 Query DSL 的搜索定义 and the prototype for the index request.

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| refresh                | boolean | 受影响的索引是否应该刷新？                                                                                                                                                                                                                                                                           |
| timeout                | time    | 每个批量请求等待不可用分片的时间。                                                                                                                                                                                                                                      |
| wait_for_active_shards | string  | 设置在执行重新索引操作之前必须处于活跃状态的分片副本数。默认为 1，即仅主分片。设为 `all` 表示所有分片副本，否则设为小于或等于分片总副本数（副本数 + 1）的任意非负值 |
| wait_for_completion    | boolean | 请求是否应阻塞直到 reindex 完成。                                                                                                                                                                                                                                                      |
| requests_per_second    | number  | 在此请求上设置的节流值，单位为每秒子请求数。-1 表示不节流。 |
| scroll                 | time    | 控制搜索上下文的保持时间 |
| slices                 | number\|string | 该任务应被分成的切片数。默认为 1（不拆分子任务），可设为 `auto`。 |
| max_docs               | number  | 要处理的最大文档数（默认：所有文档） |

## reindex_rethrottle

修改 reindex 操作的每秒请求数。

```
POST _reindex/{task_id}/_rethrottle
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------ | :----- | :------------------------------------------------------------------------------------------------- |
| requests_per_second | number | 此请求的节流值（每秒浮点子请求数）。-1 表示不节流。 |

## rollup.create

创建一个 rollup 任务。Rollup 任务可以将历史数据按时间维度聚合压缩，减少存储开销。

```
PUT _rollup/jobs/{rollupID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| rollupID       | string   | 必需。Rollup 任务的唯一标识符。                                           |
| if_seq_no      | number   | 仅在文档序列号匹配时执行操作，用于乐观并发控制。                             |
| if_primary_term| number   | 仅在文档主要任期匹配时执行操作，用于乐观并发控制。                           |
| replace        | boolean  | 是否替换已有的同名 rollup 任务。默认为 `false`。                           |
| refresh        | string   | 控制请求所做更改何时可被搜索到。有效值：`true`、`false`、`wait_for`。       |

#### HTTP 请求体

Rollup 任务配置，包含源索引、目标索引、聚合维度、调度策略等定义。

## rollup.delete

删除指定的 rollup 任务。

```
DELETE _rollup/jobs/{rollupID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| rollupID   | string   | 必需。要删除的 rollup 任务 ID。                                           |
| refresh    | string   | 控制请求所做更改何时可被搜索到。有效值：`true`、`false`、`wait_for`。       |

## rollup.explain

查看一个或多个 rollup 任务的运行详情和执行状态。

```
GET _rollup/jobs/{rollupID}/_explain
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| rollupID   | string   | 必需。Rollup 任务 ID，多个 ID 之间用逗号分隔。                             |

## rollup.get

获取 rollup 任务的定义信息。可以获取单个任务或列出所有任务。

```
GET _rollup/jobs
GET _rollup/jobs/{rollupID}
HEAD _rollup/jobs/{rollupID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| rollupID       | string   | Rollup 任务 ID。省略则列出所有 rollup 任务。                               |
| search         | string   | 按名称搜索 rollup 任务的过滤字符串。                                      |
| from           | number   | 分页起始位置（从 0 开始）。                                                |
| size           | number   | 每页返回的 rollup 任务数量。                                               |
| sortField      | string   | 排序字段名称。                                                             |
| sortDirection  | string   | 排序方向。有效值：`asc`、`desc`。                                          |

## rollup.search

搜索包含 rollup 数据的索引。当索引配置了 rollup 任务时，搜索请求会自动合并原始数据与 rollup 聚合数据。此 API 是对标准 `_search` 的增强，通过 `enable_rollup` 参数控制是否启用 rollup 搜索。

```
GET {index}/_search
POST {index}/_search
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| enable_rollup  | boolean  | 是否启用 rollup 搜索。默认为 `true`。设为 `false` 时退化为标准搜索。       |
| debug          | boolean  | 是否启用调试模式，输出更多日志信息。默认为 `false`。                       |

#### HTTP 请求体

标准搜索请求体。当启用 rollup 搜索时，查询必须包含聚合（aggregations），且只能指定单个索引，`size` 必须为 0。

## rollup.start

启动已停止的 rollup 任务。

```
POST _rollup/jobs/{rollupID}/_start
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| rollupID   | string   | 必需。要启动的 rollup 任务 ID。                                           |

## rollup.stop

停止正在运行的 rollup 任务。

```
POST _rollup/jobs/{rollupID}/_stop
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| rollupID   | string   | 必需。要停止的 rollup 任务 ID。                                           |

## rollup.update

更新已有的 rollup 任务的部分配置（如执行间隔、分页大小等）。

```
POST _rollup/jobs/{rollupID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| rollupID   | string   | 必需。要更新的 rollup 任务 ID。                                           |

#### HTTP 请求体

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| interval   | string   | 新的执行间隔。                                                             |
| page_size  | number   | 新的分页大小。                                                             |

## render_search_template

使用 Mustache 模板语言预渲染搜索定义。

```
GET _render/template
POST _render/template
```

```
GET _render/template/{id}
POST _render/template/{id}
```

#### HTTP 请求体

搜索定义模板及其参数

## scripts_painless_execute

执行任意 Painless 脚本并返回结果。

```
GET _scripts/painless/_execute
POST _scripts/painless/_execute
```

#### HTTP 请求体

要执行的脚本

## scroll

通过滚动方式从单次搜索请求中获取大量结果。

```
GET _search/scroll
POST _search/scroll
```

```
GET _search/scroll/{scroll_id}
POST _search/scroll/{scroll_id}
```

#### HTTP 请求体

如未通过 URL 或查询参数传递的滚动 ID。

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :----------------------------------------------------------------------------------------------------- |
| scroll                 | time    | 指定滚动搜索的索引一致性视图应维持多长时间               |
| scroll_id              | string  | 滚动搜索的滚动 ID |
| rest_total_hits_as_int | boolean | 指示 hits.total 在搜索响应中是渲染为整数还是对象 |

## search

执行搜索查询，返回匹配的文档。

```
GET _search
POST _search
```

```
GET {index}/_search
POST {index}/_search
```

```
GET {index}/{type}/_search
POST {index}/{type}/_search
```

#### HTTP 请求体

使用 Query DSL 的搜索定义

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| analyzer                      | string  | 用于查询字符串的分析器                                                                                                                                                                                                                                                                                                                                                                                         |
| analyze_wildcard              | boolean | 是否分析通配符和前缀查询（默认：false）                                                                                                                                                                                                                                                                                                                                                  |
| ccs_minimize_roundtrips       | boolean | 指示是否应在跨集群搜索请求执行过程中最小化网络往返 |
| default_operator              | enum    | 查询字符串查询的默认运算符（AND 或 OR）                                                                                                                                                                                                                                                                                                                                                                          |
| df                            | string  | 查询字符串中未指定字段前缀时使用的默认字段 |
| explain                       | boolean | 指定是否返回命中结果中分数计算的详细信息 |
| stored_fields                 | list    | 要作为命中结果的一部分返回的存储字段，逗号分隔                                                                                                                                                                                                                                                                                                                                                               |
| docvalue_fields               | list    | 要作为 docvalue 表示返回的字段列表（逗号分隔），用于每个命中结果                                                                                                                                                                                                                                                                                                                                |
| from                          | number  | 起始偏移量（默认：0） |
| ignore_unavailable            | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                                                                                                                                                                                                                                                                                        |
| ignore_throttled              | boolean | 当被节流时，是否忽略指定的具体、扩展或别名索引                                                                                                                                                                                                                                                                                                                                         |
| allow_no_indices              | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。                                                                                                                                                                                                                                                                       |
| expand_wildcards              | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                                                                                                                                                                                                                                                                                         |
| lenient                       | boolean | 是否忽略基于格式的查询失败（如向数字字段提供文本）                                                                                                                                                                                                                                                                                                                        |
| preference                    | string  | 指定操作应在哪个节点或分片上执行（默认：随机）                                                                                                                                                                                                                                                                                                                                                 |
| q                             | string  | Lucene 查询字符串语法的查询                                                                                                                                                                                                                                                                                                                                                                                          |
| routing                       | list    | 指定路由值列表，逗号分隔                                                                                                                                                                                                                                                                                                                                                                                |
| scroll                        | time    | 指定滚动搜索的索引一致性视图应维持多长时间                                                                                                                                                                                                                                                                                                                                         |
| search_type                   | enum    | 搜索操作类型                                                                                                                                                                                                                                                                                                                                                                                                            |
| size                          | number  | 要返回的命中数（默认：10） |
| sort                          | list    | 逗号分隔的 <field>:<direction> 对列表 |
| \_source                      | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes             | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes             | list    | 要从 \_source 字段中提取并返回的字段列表 |
| terminate_after               | number  | 每个分片收集的最大文档数，达到此数量后查询将提前终止。                                                                                                                                                                                                                                                                                                         |
| stats                         | list    | 请求的特定 'tag'，用于日志和统计目的 |
| suggest_field                 | string  | 指定用于建议的字段 |
| suggest_mode                  | enum    | 指定建议模式                                                                                                                                                                                                                                                                                                                                                                                                             |
| suggest_size                  | number  | 响应中返回多少个建议                                                                                                                                                                                                                                                                                                                                                                                       |
| suggest_text                  | string  | 要返回建议的源文本                                                                                                                                                                                                                                                                                                                                                                     |
| timeout                       | time    | 显式操作超时时间 |
| track_scores                  | boolean | 是否计算并返回分数，即使它们不用于排序                                                                                                                                                                                                                                                                                                                                                     |
| track_total_hits              | boolean | 指示是否应跟踪匹配查询的文档数量 |
| allow_partial_search_results  | boolean | 指示在部分搜索失败或超时时是否应返回错误 |
| typed_keys                    | boolean | 指定响应中聚合和建议器名称是否应以其各自类型作为前缀 |
| version                       | boolean | 是否在命中结果中返回文档版本号                                                                                                                                                                                                                                                                                                                                                                      |
| seq_no_primary_term           | boolean | 指定是否返回每个命中结果最后修改的序列号和主要任期 |
| request_cache                 | boolean | 指定此请求是否使用请求缓存，默认使用索引级别设置 |
| batched_reduce_size           | number  | 协调节点上一次应归约的分片结果数量。当请求涉及的潜在分片数量较大时，此值可作为保护机制减少每个搜索请求的内存开销。                                                                                                                                                                         |
| max_concurrent_shard_requests | number  | 此搜索在每个节点上并发执行的并发分片请求数。此值用于限制搜索对集群的影响，以限制并发分片请求的数量 |
| pre_filter_shard_size         | number  | 预过滤阈值。当搜索请求扩展到的分片数超过此阈值时，会强制执行基于查询重写的预过滤来筛选搜索分片。例如，如果分片根据其重写方法无法匹配任何文档（如日期过滤器为必需但分片范围与查询不相交），此轮预过滤可显著减少分片数量。 |
| rest_total_hits_as_int        | boolean | 指示 hits.total 在搜索响应中是渲染为整数还是对象                                                                                                                                                                                                                                                                                                                           |


## search_pipeline.delete

删除指定的搜索管道。

```
DELETE _search/pipeline/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| master_timeout | time | 指定连接主节点的超时时间 |
| timeout        | time | 显式操作超时时间 |

## search_pipeline.get

获取一个或多个搜索管道的定义。不指定 ID 时返回所有管道。

```
GET _search/pipeline
```

```
GET _search/pipeline/{id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| master_timeout | time | 指定连接主节点的超时时间 |

## search_pipeline.put

创建或更新搜索管道。如果管道的请求处理器中包含 `semantic_query_enricher`，其 `api_key` 字段会在存储前自动加密。

```
PUT _search/pipeline/{id}
```

#### HTTP 请求体

搜索管道的配置定义（包括处理器等）。

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :--------------------------------------- |
| master_timeout | time | 指定连接主节点的超时时间 |
| timeout        | time | 显式操作超时时间 |

## search_shards

返回搜索请求将要执行的索引和分片信息。

```
GET _search_shards
POST _search_shards
```

```
GET {index}/_search_shards
POST {index}/_search_shards
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| preference         | string  | 指定操作应在哪个节点或分片上执行（默认：随机）                                                                           |
| routing            | string  | 特定的路由值 |
| local              | boolean | 返回本地信息，不从主节点获取状态（默认：false）                                                                      |
| ignore_unavailable | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| allow_no_indices   | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards   | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |

## search_template

使用 Mustache 模板语言预渲染搜索定义。

```
GET _search/template
POST _search/template
```

```
GET {index}/_search/template
POST {index}/_search/template
```

```
GET {index}/{type}/_search/template
POST {index}/{type}/_search/template
```

#### HTTP 请求体

搜索定义模板及其参数

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :---------------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ignore_unavailable      | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                  |
| ignore_throttled        | boolean | 当被节流时，是否忽略指定的具体、扩展或别名索引                                                                   |
| allow_no_indices        | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。 |
| expand_wildcards        | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                   |
| preference              | string  | 指定操作应在哪个节点或分片上执行（默认：随机）                                                                           |
| routing                 | list    | 指定路由值列表，逗号分隔                                                                                                          |
| scroll                  | time    | 指定滚动搜索的索引一致性视图应维持多长时间                                                                   |
| search_type             | enum    | 搜索操作类型                                                                                                                                      |
| explain                 | boolean | 指定是否返回命中结果中分数计算的详细信息 |
| profile                 | boolean | 指定是否对查询执行进行性能分析 |
| typed_keys              | boolean | 指定响应中聚合和建议器名称是否应以其各自类型作为前缀 |
| rest_total_hits_as_int  | boolean | 指示 hits.total 在搜索响应中是渲染为整数还是对象                                                     |
| ccs_minimize_roundtrips | boolean | 指示是否应在跨集群搜索请求执行过程中最小化网络往返 |

## security.account

获取或更新当前登录用户的账户信息，包括密码修改。

```
GET _security/account
PUT _security/account
```

#### HTTP 请求体（PUT）

用户账户更新信息，如新密码等。

#### GET 响应字段

| 字段 | 类型 | 说明 |
| :------------------ | :------------- | :------------------------------------------------------------- |
| username            | string         | 当前登录用户名 |
| reserved            | boolean        | 是否为保留用户 |
| hidden              | boolean        | 是否为隐藏用户 |
| builtin             | boolean        | 是否为内置用户 |
| external_roles      | array[string]  | 当前用户的外部角色 |
| attributes          | array[string]  | 当前用户的自定义属性键 |
| roles               | array[string]  | 最终映射得到的安全角色 |
| cluster_privileges  | array[string]  | 聚合后的集群级权限（多角色去重合并；即使包含 `*` 也会保留其他具体项） |
| index_privileges    | array[object]  | 按角色展开的索引级权限列表 |

`index_privileges` 每个对象包含：

| 字段 | 类型 | 说明 |
| :----------- | :------------- | :------------------------------ |
| role         | string         | 权限来源角色名 |
| names        | array[string]  | 索引名称或模式 |
| privileges   | array[string]  | 该角色在对应索引上的权限 |

## security.authinfo

获取当前经过身份验证的用户信息，包括角色、后端角色等。

```
GET _security/authinfo
POST _security/authinfo
```

## security.authtoken

生成身份验证令牌。

```
POST _security/authtoken
```

#### HTTP 请求体

令牌请求参数。

## security.cache

刷新安全模块缓存，使安全配置更改立即生效。

```
DELETE _security/cache
GET _security/cache
PUT _security/cache
POST _security/cache
```

## security.config

获取或更新安全模块的全局配置。

```
GET _security/securityconfig/
PUT _security/securityconfig/{name}
PATCH _security/securityconfig/
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| name     | string   | 配置名称（用于 PUT 操作）。                                               |

#### HTTP 请求体（PUT/PATCH）

安全配置内容。

## security.nodesdn

管理可信节点的 DN（Distinguished Name）列表，用于节点间 TLS 通信认证。

```
GET _security/nodesdn/
GET _security/nodesdn/{name}
DELETE _security/nodesdn/{name}
PUT _security/nodesdn/{name}
PATCH _security/nodesdn/
PATCH _security/nodesdn/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| name     | string   | 节点 DN 条目名称。省略则列出所有条目。                                     |

#### HTTP 请求体（PUT/PATCH）

节点 DN 配置内容。

## security.permissionsinfo

获取当前用户的权限信息。

```
GET _security/permissionsinfo
```

## security.privilege

管理操作组（权限组），用于将多个权限打包为一个命名集合。

```
GET _security/privilege/
GET _security/privilege/{name}
DELETE _security/privilege/{name}
PUT _security/privilege/{name}
PATCH _security/privilege/
PATCH _security/privilege/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| name     | string   | 操作组名称。省略则列出所有操作组。                                         |

#### HTTP 请求体（PUT/PATCH）

操作组定义，包含允许的操作列表。

## security.role

管理安全角色。角色定义了用户可以访问的集群权限和索引权限。

```
GET _security/role/
GET _security/role/{name}
DELETE _security/role/{name}
PUT _security/role/{name}
PATCH _security/role/
PATCH _security/role/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| name     | string   | 角色名称。省略则列出所有角色。                                             |

#### HTTP 请求体（PUT/PATCH）

角色定义，包含集群权限、索引权限、租户权限等。

## security.role_mapping

管理角色映射，将后端角色、用户等映射到安全角色。

```
GET _security/role_mapping/
GET _security/role_mapping/{name}
DELETE _security/role_mapping/{name}
PUT _security/role_mapping/{name}
PATCH _security/role_mapping/
PATCH _security/role_mapping/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| name     | string   | 角色映射名称。省略则列出所有映射。                                         |

#### HTTP 请求体（PUT/PATCH）

角色映射定义，包含后端角色、用户、主机列表等。

## security.ssl_certificates

获取当前节点正在使用的 SSL/TLS 证书摘要信息（ES 兼容格式）。

```
GET _security/ssl/certificates
```

#### 权限要求

- 需要 `monitor` 或 `superuser` 角色。

#### 响应说明

返回数组，每个元素代表一个证书条目（HTTP 和 Transport）：

| 字段 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| path | string | 证书文件路径。返回相对 `config` 目录的路径，包含真实文件名（例如 `instance.crt`）。 |
| format | string | 证书格式，常见值：`PEM`、`JKS`、`PKCS12`。 |
| alias | string | 证书别名。 |
| subject_dn | string | 证书 Subject DN。 |
| serial_number | string | 证书序列号（十六进制字符串）。 |
| has_private_key | boolean | 是否包含私钥。 |
| expiry | string | 证书过期时间（UTC 时间戳）。 |

#### 失败返回

- `403 Forbidden`：当前用户无权限访问该 API。
- `500 Internal Server Error`：证书路径未配置或读取失败。

## security.ssl_certs

获取当前节点 SSL/TLS 证书明细信息（包含颁发者、SAN、有效期等）。

```
GET _security/ssl/certs
```

#### 权限要求

- 仅允许 `admin_dn`（管理员证书）访问。
- 使用普通 Basic Auth 用户（即使用户名为 `admin`）通常会被拒绝。

#### 响应说明

返回对象，包含两个列表：

| 字段 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| http_certificates_list | array | HTTP 证书明细列表。 |
| transport_certificates_list | array | Transport 证书明细列表。 |

证书明细项字段：

| 字段 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| issuer_dn | string | 颁发者 DN。 |
| subject_dn | string | 主题 DN。 |
| san | string | Subject Alternative Names。 |
| not_before | string | 生效时间（UTC）。 |
| not_after | string | 过期时间（UTC）。 |

#### 失败返回

- `403 Forbidden`：无权限访问该 API（标准错误结构）。

## security.ssl_reload_certs

重新加载指定类型的 SSL/TLS 证书，无需重启节点。

```
PUT _security/ssl/{certType}/reloadcerts/
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| certType   | string   | 必需。证书类型。有效值：`http`、`transport`。                              |

## security.sslinfo

获取节点的 SSL/TLS 配置信息。

```
GET _security/sslinfo
```

## security.user

管理内部用户。支持创建、读取、更新和删除用户。

```
GET _security/user/
GET _security/user/{name}
DELETE _security/user/{name}
PUT _security/user/{name}
PATCH _security/user/
PATCH _security/user/{name}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------- | :------- | :----------------------------------------------------------------------- |
| name     | string   | 用户名。省略则列出所有用户。                                               |

#### HTTP 请求体（PUT/PATCH）

用户定义，包含密码、后端角色、属性等。

## security.validate

验证安全模块配置的有效性。

```
GET _security/validate
```

## security.whitelist

管理 API 白名单。控制哪些 REST API 端点可以被访问。

```
GET _security/whitelist
PUT _security/whitelist
PATCH _security/whitelist
```

#### HTTP 请求体（PUT/PATCH）

白名单配置，包含允许的 API 端点列表。

## snapshot.cleanup_repository

清理快照仓库中未被引用的过期数据。

```
POST _snapshot/{repository}/_cleanup
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :------------------------------------------------------- |
| master_timeout | time | 连接主节点的显式超时时间 |
| timeout        | time | 显式操作超时时间 |

## snapshot.clone

在同一仓库内将快照中的索引克隆到另一个快照。

```
PUT _snapshot/{repository}/{snapshot}/_clone/{target_snapshot}
```

#### HTTP 请求体

快照克隆定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :------------------------------------------------------- |
| master_timeout | time | 连接主节点的显式超时时间 |

## snapshot.create

在仓库中创建快照。

```
PUT _snapshot/{repository}/{snapshot}
POST _snapshot/{repository}/{snapshot}
```

#### HTTP 请求体

快照定义

**必需**：否

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------ | :------ | :-------------------------------------------------------------------------- |
| master_timeout      | time    | 连接主节点的显式超时时间                    |
| wait_for_completion | boolean | 此请求是否应等待操作完成后再返回 |

## snapshot.create_repository

创建快照仓库。

```
PUT _snapshot/{repository}
POST _snapshot/{repository}
```

#### HTTP 请求体

仓库定义

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------- |
| master_timeout | time    | 连接主节点的显式超时时间 |
| timeout        | time    | 显式操作超时时间 |
| verify         | boolean | 创建后是否验证仓库          |

## snapshot.delete

删除快照。

```
DELETE _snapshot/{repository}/{snapshot}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :------------------------------------------------------- |
| master_timeout | time | 连接主节点的显式超时时间 |

## snapshot.delete_repository

删除快照仓库。

```
DELETE _snapshot/{repository}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :------------------------------------------------------- |
| master_timeout | time | 连接主节点的显式超时时间 |
| timeout        | time | 显式操作超时时间 |

## snapshot.get

返回快照的信息。

```
GET _snapshot/{repository}/{snapshot}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :---------------------------------------------------------------------------------------------------------- |
| master_timeout     | time    | 连接主节点的显式超时时间                                                    |
| ignore_unavailable | boolean | 是否忽略不可用的快照，默认为 false，即会抛出 SnapshotMissingException |
| verbose            | boolean | 是否显示详细快照信息，还是仅显示仓库索引 blob 中的基本信息 |

## snapshot.get_repository

返回快照仓库的信息。

```
GET _snapshot
```

```
GET _snapshot/{repository}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------ | :------------------------------------------------------------------------------------ |
| master_timeout | time    | 连接主节点的显式超时时间                              |
| local          | boolean | 返回本地信息，不从主节点获取状态（默认：false） |

## snapshot.restore

从快照恢复数据。

```
POST _snapshot/{repository}/{snapshot}/_restore
```

#### HTTP 请求体

要恢复的详细信息

**必需**：否

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------ | :------ | :-------------------------------------------------------------------------- |
| master_timeout      | time    | 连接主节点的显式超时时间                    |
| wait_for_completion | boolean | 此请求是否应等待操作完成后再返回 |

## snapshot.status

返回快照的详细状态信息。

```
GET _snapshot/_status
```

```
GET _snapshot/{repository}/_status
```

```
GET _snapshot/{repository}/{snapshot}/_status
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :----------------- | :------ | :---------------------------------------------------------------------------------------------------------- |
| master_timeout     | time    | 连接主节点的显式超时时间                                                    |
| ignore_unavailable | boolean | 是否忽略不可用的快照，默认为 false，即会抛出 SnapshotMissingException |

## snapshot.verify_repository

验证快照仓库的可用性。

```
POST _snapshot/{repository}/_verify
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :--- | :------------------------------------------------------- |
| master_timeout | time | 连接主节点的显式超时时间 |
| timeout        | time | 显式操作超时时间 |

## sql.close_cursor

关闭 SQL 查询的游标，释放服务端资源。

```
POST _sql/close
```

#### HTTP 请求体

包含要关闭的游标 ID。

## sql.explain

对 SQL 查询语句进行解析，返回对应的 Easysearch 查询 DSL，而不实际执行查询。

```
POST _sql/_explain
```

#### HTTP 请求体

SQL 查询语句。

## sql.query

执行 SQL 查询并返回结果。支持 JDBC 格式输出。

```
POST _sql
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------- | :------- | :----------------------------------------------------------------------- |
| format     | string   | 结果格式，如 `jdbc`、`csv`、`json` 等。                                   |
| flat       | string   | 是否展平结果。                                                             |
| separator  | string   | CSV 输出分隔符。                                                           |

#### HTTP 请求体

SQL 查询语句及参数。

## sql.settings

更新 SQL 插件的查询设置。

```
PUT _sql/_query/settings
```

#### HTTP 请求体

| 参数 | 类型 | 说明 |
| :---------- | :------- | :----------------------------------------------------------------------- |
| transient   | object   | 临时 SQL 设置，节点重启后失效。                                            |
| persistent  | object   | 持久 SQL 设置，节点重启后仍保留。                                          |

## sql.stats

获取 SQL 插件的统计指标。

```
POST _sql/stats
```

## tasks.cancel

取消正在执行的任务（如果该任务支持通过 API 取消）。

```
POST _tasks/_cancel
```

```
POST _tasks/{task_id}/_cancel
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------ | :------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| nodes               | list    | 用于限制返回信息的节点 ID 或名称列表（逗号分隔）；使用 `_local` 返回当前连接节点的信息，留空获取所有节点的信息 |
| actions             | list    | 要取消的操作列表（逗号分隔）。留空表示取消全部。                                                                                                              |
| parent_task_id      | string  | 取消指定父任务 ID 的任务（node_id:task_number）。设为 -1 取消全部。                                                                                                          |
| wait_for_completion | boolean | 请求是否应阻塞直到任务及其子任务的取消完成。默认为 false |

## tasks.get

返回指定任务的详细信息。

```
GET _tasks/{task_id}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------ | :------ | :------------------------------------------------------- |
| wait_for_completion | boolean | 等待匹配的任务完成（默认：false） |
| timeout             | time    | 显式操作超时时间 |

## tasks.list

返回正在执行的任务列表。

```
GET _tasks
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------ | :------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| nodes               | list    | 用于限制返回信息的节点 ID 或名称列表（逗号分隔）；使用 `_local` 返回当前连接节点的信息，留空获取所有节点的信息 |
| actions             | list    | 要返回的操作列表（逗号分隔）。留空返回全部。                                                                                                               |
| detailed            | boolean | 返回详细的任务信息（默认：false） |
| parent_task_id      | string  | 返回指定父任务 ID 的任务（node_id:task_number）。设为 -1 返回全部。                                                                                                          |
| wait_for_completion | boolean | 等待匹配的任务完成（默认：false）                                                                                                                                            |
| group_by            | enum    | 按节点或父/子关系对任务进行分组                                                                                                                                                  |
| timeout             | time    | 显式操作超时时间 |

## termvectors

返回指定文档字段中词项的信息和统计数据。

```
GET {index}/_termvectors/{id}
POST {index}/_termvectors/{id}
```

```
GET {index}/_termvectors
POST {index}/_termvectors
```

```
GET {index}/{type}/{id}/_termvectors
POST {index}/{type}/{id}/_termvectors
```

```
GET {index}/{type}/_termvectors
POST {index}/{type}/_termvectors
```

#### HTTP 请求体

定义参数和/或提供文档以获取词条向量。详见文档。

**必需**：否

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------- | :------ | :------------------------------------------------------------------------------------------------------------- |
| term_statistics  | boolean | 指定是否返回词条总频率和文档频率。 |
| field_statistics | boolean | 指定是否返回文档计数、文档频率总和以及词条总频率总和。 |
| fields           | list    | 逗号分隔的要返回的字段列表。 |
| offsets          | boolean | 指定是否返回词条偏移量。 |
| positions        | boolean | 指定是否返回词条位置。 |
| payloads         | boolean | 指定是否返回词条负载。 |
| preference       | string  | 指定操作应在哪个节点或分片上执行（默认：随机）.                              |
| routing          | string  | 指定路由值。                                                                                        |
| realtime         | boolean | 指定请求是否为实时而非近实时（默认：true）。 |
| version          | number  | 用于并发控制的显式版本号                                                                |
| version_type     | enum    | 指定版本类型                                                                                          |

<!--
## transform.create

创建一个 transform（数据转换）任务。Transform 可以将源索引的数据通过聚合转换后写入目标索引。

```
PUT _ilm/_transform/{transformID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| transformID    | string   | 必需。Transform 任务的唯一标识符。                                         |
| if_seq_no      | number   | 仅在序列号匹配时执行操作，用于乐观并发控制。                                 |
| if_primary_term| number   | 仅在主要任期匹配时执行操作，用于乐观并发控制。                               |
| refresh        | string   | 控制请求所做更改何时可被搜索到。有效值：`true`、`false`、`wait_for`。       |

#### HTTP 请求体

Transform 任务配置，包含源索引、目标索引、聚合定义、分组字段等。

## transform.delete

删除一个或多个 transform 任务。

```
DELETE _ilm/_transform/{transformID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| transformID    | string   | 必需。以逗号分隔的 transform 任务 ID。                                     |
| force          | boolean  | 是否强制删除正在运行的 transform 任务。                                    |

## transform.explain

查看一个或多个 transform 任务的运行状态和执行详情。

```
GET _ilm/_transform/{transformID}/_explain
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| transformID    | string   | 必需。以逗号分隔的 transform 任务 ID。                                     |

## transform.get

获取 transform 任务的定义信息。可以获取单个任务或列出所有任务。

```
GET _ilm/_transform
GET _ilm/_transform/{transformID}
HEAD _ilm/_transform/{transformID}
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| transformID    | string   | Transform 任务 ID。省略则列出所有任务。                                    |
| search         | string   | 按名称搜索 transform 任务的过滤字符串。                                    |
| from           | number   | 分页起始位置（从 0 开始）。                                                |
| size           | number   | 每页返回数量。                                                             |
| sortField      | string   | 排序字段名称。                                                             |
| sortDirection  | string   | 排序方向。有效值：`asc`、`desc`。                                          |

## transform.preview

预览 transform 任务的输出结果，不实际创建目标索引。

```
POST _ilm/_transform
POST _ilm/_transform/_preview
```

#### HTTP 请求体

Transform 任务配置，与 `transform.create` 相同。

## transform.start

启动已停止的 transform 任务。

```
POST _ilm/_transform/{transformID}/_start
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| transformID    | string   | 必需。要启动的 transform 任务 ID。                                         |

## transform.stop

停止正在运行的 transform 任务。

```
POST _ilm/_transform/{transformID}/_stop
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------- | :------- | :----------------------------------------------------------------------- |
| transformID    | string   | 必需。要停止的 transform 任务 ID。                                         |
-->

## update

使用脚本或部分文档更新文档。

```
POST {index}/_update/{id}
```

```
POST {index}/{type}/{id}/_update
```

#### HTTP 请求体

请求定义需要 `script` 或部分 `doc`

**必需**：是

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| wait_for_active_shards | string  | 设置在执行更新操作之前必须处于活跃状态的分片副本数。默认为 1，即仅主分片。设为 `all` 表示所有分片副本，否则设为小于或等于分片总副本数（副本数 + 1）的任意非负值 |
| \_source               | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes      | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes      | list    | 要从 \_source 字段中提取并返回的字段列表 |
| lang                   | string  | 脚本语言（默认：painless） |
| refresh                | enum    | 如果为 `true`，刷新受影响的分片使操作对搜索可见；如果为 `wait_for`，等待刷新完成后再返回；如果为 `false`（默认），不执行刷新操作。                                                                                      |
| retry_on_conflict      | number  | 指定发生冲突时操作应重试的次数（默认：0） |
| routing                | string  | 特定的路由值 |
| timeout                | time    | 显式操作超时时间 |
| if_seq_no              | number  | 仅在更改文档的最后操作具有指定序列号时才执行更新操作 |
| if_primary_term        | number  | 仅在更改文档的最后操作具有指定主要任期时才执行更新操作 |
| require_alias          | boolean | 当为 true 时，要求目标必须是别名。默认为 false |

## update_by_query

对索引中的每个文档执行更新而不更改源数据，
例如以应用映射变更。

```
POST {index}/_update_by_query
```

```
POST {index}/{type}/_update_by_query
```

#### HTTP 请求体

使用 Query DSL 的搜索定义

#### URL 参数

| 参数 | 类型 | 说明 |
| :--------------------- | :------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| analyzer               | string  | 用于查询字符串的分析器                                                                                                                                                                                                                                                                                    |
| analyze_wildcard       | boolean | 是否分析通配符和前缀查询（默认：false）                                                                                                                                                                                                                                             |
| default_operator       | enum    | 查询字符串查询的默认运算符（AND 或 OR）                                                                                                                                                                                                                                                                     |
| df                     | string  | 查询字符串中未指定字段前缀时使用的默认字段 |
| from                   | number  | 起始偏移量（默认：0） |
| ignore_unavailable     | boolean | 当指定的具体索引不可用（缺失或已关闭）时是否忽略                                                                                                                                                                                                                                   |
| allow_no_indices       | boolean | 当通配符表达式不匹配任何具体索引时是否忽略（包括 `_all` 或未指定索引的情况）。                                                                                                                                                                  |
| conflicts              | enum    | 当 update-by-query 遇到版本冲突时的行为                                                                                                                                                                                                                                                                 |
| expand_wildcards       | enum    | 是否将通配符表达式扩展为打开的、关闭的或全部索引。                                                                                                                                                                                                                                    |
| lenient                | boolean | 是否忽略基于格式的查询失败（如向数字字段提供文本）                                                                                                                                                                                                                   |
| pipeline               | string  | 在此操作发出的索引请求上设置的摄取管道。（默认：无） |
| preference             | string  | 指定操作应在哪个节点或分片上执行（默认：随机）                                                                                                                                                                                                                                            |
| q                      | string  | Lucene 查询字符串语法的查询                                                                                                                                                                                                                                                                                     |
| routing                | list    | 指定路由值列表，逗号分隔                                                                                                                                                                                                                                                                           |
| scroll                 | time    | 指定滚动搜索的索引一致性视图应维持多长时间                                                                                                                                                                                                                                    |
| search_type            | enum    | 搜索操作类型                                                                                                                                                                                                                                                                                                       |
| search_timeout         | time    | 每个搜索请求的显式超时时间。默认无超时。 |
| size                   | number  | 已弃用，请改用 `max_docs` |
| max_docs               | number  | 要处理的最大文档数（默认：所有文档） |
| sort                   | list    | 逗号分隔的 <field>:<direction> 对列表 |
| \_source               | list    | 是否返回 \_source 字段，或要返回的字段列表 |
| \_source_excludes      | list    | 要从返回的 \_source 字段中排除的字段列表 |
| \_source_includes      | list    | 要从 \_source 字段中提取并返回的字段列表 |
| terminate_after        | number  | 每个分片收集的最大文档数，达到此数量后查询将提前终止。                                                                                                                                                                                                    |
| stats                  | list    | 请求的特定 'tag'，用于日志和统计目的 |
| version                | boolean | 是否在命中结果中返回文档版本号                                                                                                                                                                                                                                                                 |
| version_type           | boolean | 文档命中时是否应增加版本号（内部）（重新索引时） |
| request_cache          | boolean | 指定此请求是否使用请求缓存，默认使用索引级别设置 |
| refresh                | boolean | 受影响的索引是否应该刷新？                                                                                                                                                                                                                                                                                   |
| timeout                | time    | 每个批量请求等待不可用分片的时间。                                                                                                                                                                                                                                              |
| wait_for_active_shards | string  | 设置在执行按查询更新操作之前必须处于活跃状态的分片副本数。默认为 1，即仅主分片。设为 `all` 表示所有分片副本，否则设为小于或等于分片总副本数（副本数 + 1）的任意非负值 |
| scroll_size            | number  | 驱动 update-by-query 的滚动请求大小                                                                                                                                                                                                                                                                     |
| wait_for_completion    | boolean | 请求是否应阻塞直到按查询更新操作完成。 |
| requests_per_second    | number  | 在此请求上设置的节流值，单位为每秒子请求数。-1 表示不节流。 |
| slices                 | number\|string | 此任务应被分成的切片数。默认为 1，即任务不会被分成子任务。可以设为 `auto`。 |

## update_by_query_rethrottle

修改 update-by-query 操作的每秒请求数。

```
POST _update_by_query/{task_id}/_rethrottle
```

#### URL 参数

| 参数 | 类型 | 说明 |
| :------------------ | :----- | :------------------------------------------------------------------------------------------------- |
| requests_per_second | number | 此请求的节流值（每秒浮点子请求数）。-1 表示不节流。 |

