---
title: "权限列表"
date: 0001-01-01
summary: "权限列表 #  此页面是可用权限的完整列表。每个权限控制对数据类型或 API 的访问。
集群权限 #  核心集群权限 #   cluster:admin/component_template/delete cluster:admin/component_template/get cluster:admin/component_template/put cluster:admin/indices/dangling/delete cluster:admin/indices/dangling/find cluster:admin/indices/dangling/import cluster:admin/indices/dangling/list cluster:admin/ingest/pipeline/delete cluster:admin/ingest/pipeline/get cluster:admin/ingest/pipeline/put cluster:admin/ingest/pipeline/simulate cluster:admin/ingest/processor/grok/get cluster:admin/nodes/reload_secure_settings cluster:admin/reindex/rethrottle cluster:admin/repository/_cleanup cluster:admin/repository/delete cluster:admin/repository/get cluster:admin/repository/put cluster:admin/repository/verify cluster:admin/reroute cluster:admin/script/delete cluster:admin/script/get cluster:admin/script/put cluster:admin/script_context/get cluster:admin/script_language/get cluster:admin/scripts/painless/context cluster:admin/scripts/painless/execute cluster:admin/search/pipeline/delete cluster:admin/search/pipeline/get cluster:admin/search/pipeline/put cluster:admin/settings/update cluster:admin/snapshot/clone cluster:admin/snapshot/create cluster:admin/snapshot/delete cluster:admin/snapshot/get cluster:admin/snapshot/restore cluster:admin/snapshot/status cluster:admin/snapshot/status* cluster:admin/tasks/cancel cluster:admin/tasks/test cluster:admin/tasks/testunblock cluster:admin/voting_config/add_exclusions cluster:admin/voting_config/clear_exclusions cluster:monitor/allocation/explain cluster:monitor/health cluster:monitor/main cluster:monitor/nodes/hot_threads cluster:monitor/nodes/info cluster:monitor/nodes/liveness cluster:monitor/nodes/stats cluster:monitor/nodes/usage cluster:monitor/remote/info cluster:monitor/state cluster:monitor/stats cluster:monitor/task cluster:monitor/task/get cluster:monitor/tasks/lists  异步搜索权限 #   cluster:admin/async_search/delete cluster:admin/async_search/get cluster:admin/async_search/stats cluster:admin/async_search/submit  安全管理权限 #   cluster:admin/security/config/update cluster:admin/security/whoami  ILM 索引生命周期管理权限 #   cluster:admin/ilm/managedindex/add cluster:admin/ilm/managedindex/change cluster:admin/ilm/managedindex/explain cluster:admin/ilm/managedindex/remove cluster:admin/ilm/managedindex/retry cluster:admin/ilm/policy/delete cluster:admin/ilm/policy/get cluster:admin/ilm/policy/search cluster:admin/ilm/policy/write cluster:admin/ilm/update/managedindexmetadata  Rollup 权限 #   cluster:admin/rollup/delete cluster:admin/rollup/explain cluster:admin/rollup/get cluster:admin/rollup/index cluster:admin/rollup/mapping/update cluster:admin/rollup/search cluster:admin/rollup/start cluster:admin/rollup/stop cluster:admin/rollup/update  Transform 权限 #   cluster:admin/transform/delete cluster:admin/transform/explain cluster:admin/transform/get cluster:admin/transform/get_transforms cluster:admin/transform/index cluster:admin/transform/preview cluster:admin/transform/start cluster:admin/transform/stop  快照管理权限 #   cluster:admin/snapshot_management/policy/delete cluster:admin/snapshot_management/policy/explain cluster:admin/snapshot_management/policy/get cluster:admin/snapshot_management/policy/search cluster:admin/snapshot_management/policy/start cluster:admin/snapshot_management/policy/stop cluster:admin/snapshot_management/policy/write  通知权限 #   cluster:admin/easysearch/notifications/channels/get cluster:admin/easysearch/notifications/configs/create cluster:admin/easysearch/notifications/configs/delete cluster:admin/easysearch/notifications/configs/get cluster:admin/easysearch/notifications/configs/update cluster:admin/easysearch/notifications/feature/publish cluster:admin/easysearch/notifications/feature/send cluster:admin/easysearch/notifications/features  跨集群复制权限 #   cluster:admin/plugins/replication/autofollow cluster:admin/plugins/replication/autofollow/update cluster:indices/admin/replication cluster:indices/shards/replication  插件权限 #   cluster:admin/ai/embed cluster:admin/ik/dictionary/reload cluster:admin/rules/compile/internal cluster:admin/rules/delete/internal cluster:admin/rules/sync/block  索引权限 #  核心索引管理权限 #   indices:admin/aliases indices:admin/aliases/exists indices:admin/aliases/get indices:admin/analyze indices:admin/analyze_space_usage indices:admin/auto_create indices:admin/block/add indices:admin/cache/clear indices:admin/close indices:admin/create indices:admin/data_stream/create indices:admin/data_stream/delete indices:admin/data_stream/get indices:admin/delete indices:admin/exists indices:admin/flush indices:admin/flush* indices:admin/forcemerge indices:admin/get indices:admin/index_template/delete indices:admin/index_template/get indices:admin/index_template/put indices:admin/index_template/simulate indices:admin/index_template/simulate_index indices:admin/mapping/auto_put indices:admin/mapping/put indices:admin/mappings/fields/get indices:admin/mappings/fields/get* indices:admin/mappings/get indices:admin/open indices:admin/refresh indices:admin/refresh* indices:admin/refresh_search_analyzers indices:admin/reload indices:admin/resize indices:admin/resolve/index indices:admin/rollover indices:admin/seq_no/global_checkpoint_sync indices:admin/settings/update indices:admin/shards/search_shards indices:admin/shrink indices:admin/synced_flush indices:admin/template/delete indices:admin/template/get indices:admin/template/put indices:admin/types/exists indices:admin/upgrade indices:admin/validate/query  索引数据读取权限 #   indices:data/read/async_search indices:data/read/explain indices:data/read/field_caps indices:data/read/field_caps* indices:data/read/get indices:data/read/mget indices:data/read/mget* indices:data/read/msearch indices:data/read/msearch/template indices:data/read/mtv indices:data/read/mtv* indices:data/read/point_in_time/create indices:data/read/point_in_time/delete indices:data/read/point_in_time/readall indices:data/read/rank_eval indices:data/read/rollup/search indices:data/read/scroll indices:data/read/scroll/clear indices:data/read/search indices:data/read/search* indices:data/read/search/template indices:data/read/tv  索引数据写入权限 #   indices:data/write/bulk indices:data/write/bulk* indices:data/write/delete indices:data/write/delete/byquery indices:data/write/index indices:data/write/reindex indices:data/write/update indices:data/write/update/byquery  索引监控权限 #   indices:monitor/data_stream/stats indices:monitor/field_usage_stats indices:monitor/recovery indices:monitor/segments indices:monitor/settings/get indices:monitor/shard_stores indices:monitor/stats indices:monitor/upgrade  ILM 索引级别权限 #   indices:admin/ilm/managedindex  跨集群复制索引权限 #   indices:admin/plugins/replication/autofollow/stats indices:admin/plugins/replication/follower/stats indices:admin/plugins/replication/index/all-shards indices:admin/plugins/replication/index/pause indices:admin/plugins/replication/index/resume indices:admin/plugins/replication/index/setup/validate indices:admin/plugins/replication/index/start indices:admin/plugins/replication/index/stats indices:admin/plugins/replication/index/status_check indices:admin/plugins/replication/index/stop indices:admin/plugins/replication/index/update indices:admin/plugins/replication/index/update_metadata indices:admin/plugins/replication/resources/release indices:admin/replication/index/all_status indices:data/read/plugins/replication/changes indices:data/read/plugins/replication/file_chunk indices:data/read/plugins/replication/file_metadata indices:data/write/plugins/replication/changes  权限集合 #  为了提高权限设置的效率，我们对系统权限进行自定义管理，从而快速批量选择一组相关的权限，而不是分别选择单个权限，为了方便，系统内置了若干权限集合。"
---


# 权限列表

此页面是可用权限的完整列表。每个权限控制对数据类型或 API 的访问。

## 集群权限

### 核心集群权限

- cluster:admin/component_template/delete
- cluster:admin/component_template/get
- cluster:admin/component_template/put
- cluster:admin/indices/dangling/delete
- cluster:admin/indices/dangling/find
- cluster:admin/indices/dangling/import
- cluster:admin/indices/dangling/list
- cluster:admin/ingest/pipeline/delete
- cluster:admin/ingest/pipeline/get
- cluster:admin/ingest/pipeline/put
- cluster:admin/ingest/pipeline/simulate
- cluster:admin/ingest/processor/grok/get
- cluster:admin/nodes/reload_secure_settings
- cluster:admin/reindex/rethrottle
- cluster:admin/repository/_cleanup
- cluster:admin/repository/delete
- cluster:admin/repository/get
- cluster:admin/repository/put
- cluster:admin/repository/verify
- cluster:admin/reroute
- cluster:admin/script/delete
- cluster:admin/script/get
- cluster:admin/script/put
- cluster:admin/script_context/get
- cluster:admin/script_language/get
- cluster:admin/scripts/painless/context
- cluster:admin/scripts/painless/execute
- cluster:admin/search/pipeline/delete
- cluster:admin/search/pipeline/get
- cluster:admin/search/pipeline/put
- cluster:admin/settings/update
- cluster:admin/snapshot/clone
- cluster:admin/snapshot/create
- cluster:admin/snapshot/delete
- cluster:admin/snapshot/get
- cluster:admin/snapshot/restore
- cluster:admin/snapshot/status
- cluster:admin/snapshot/status\*
- cluster:admin/tasks/cancel
- cluster:admin/tasks/test
- cluster:admin/tasks/testunblock
- cluster:admin/voting_config/add_exclusions
- cluster:admin/voting_config/clear_exclusions
- cluster:monitor/allocation/explain
- cluster:monitor/health
- cluster:monitor/main
- cluster:monitor/nodes/hot_threads
- cluster:monitor/nodes/info
- cluster:monitor/nodes/liveness
- cluster:monitor/nodes/stats
- cluster:monitor/nodes/usage
- cluster:monitor/remote/info
- cluster:monitor/state
- cluster:monitor/stats
- cluster:monitor/task
- cluster:monitor/task/get
- cluster:monitor/tasks/lists

### 异步搜索权限

- cluster:admin/async_search/delete
- cluster:admin/async_search/get
- cluster:admin/async_search/stats
- cluster:admin/async_search/submit

### 安全管理权限

- cluster:admin/security/config/update
- cluster:admin/security/whoami

### ILM 索引生命周期管理权限

- cluster:admin/ilm/managedindex/add
- cluster:admin/ilm/managedindex/change
- cluster:admin/ilm/managedindex/explain
- cluster:admin/ilm/managedindex/remove
- cluster:admin/ilm/managedindex/retry
- cluster:admin/ilm/policy/delete
- cluster:admin/ilm/policy/get
- cluster:admin/ilm/policy/search
- cluster:admin/ilm/policy/write
- cluster:admin/ilm/update/managedindexmetadata

### Rollup 权限

- cluster:admin/rollup/delete
- cluster:admin/rollup/explain
- cluster:admin/rollup/get
- cluster:admin/rollup/index
- cluster:admin/rollup/mapping/update
- cluster:admin/rollup/search
- cluster:admin/rollup/start
- cluster:admin/rollup/stop
- cluster:admin/rollup/update

### Transform 权限

- cluster:admin/transform/delete
- cluster:admin/transform/explain
- cluster:admin/transform/get
- cluster:admin/transform/get_transforms
- cluster:admin/transform/index
- cluster:admin/transform/preview
- cluster:admin/transform/start
- cluster:admin/transform/stop

### 快照管理权限

- cluster:admin/snapshot_management/policy/delete
- cluster:admin/snapshot_management/policy/explain
- cluster:admin/snapshot_management/policy/get
- cluster:admin/snapshot_management/policy/search
- cluster:admin/snapshot_management/policy/start
- cluster:admin/snapshot_management/policy/stop
- cluster:admin/snapshot_management/policy/write

### 通知权限

- cluster:admin/easysearch/notifications/channels/get
- cluster:admin/easysearch/notifications/configs/create
- cluster:admin/easysearch/notifications/configs/delete
- cluster:admin/easysearch/notifications/configs/get
- cluster:admin/easysearch/notifications/configs/update
- cluster:admin/easysearch/notifications/feature/publish
- cluster:admin/easysearch/notifications/feature/send
- cluster:admin/easysearch/notifications/features

### 跨集群复制权限

- cluster:admin/plugins/replication/autofollow
- cluster:admin/plugins/replication/autofollow/update
- cluster:indices/admin/replication
- cluster:indices/shards/replication

### 插件权限

- cluster:admin/ai/embed
- cluster:admin/ik/dictionary/reload
- cluster:admin/rules/compile/internal
- cluster:admin/rules/delete/internal
- cluster:admin/rules/sync/block

## 索引权限

### 核心索引管理权限

- indices:admin/aliases
- indices:admin/aliases/exists
- indices:admin/aliases/get
- indices:admin/analyze
- indices:admin/analyze_space_usage
- indices:admin/auto_create
- indices:admin/block/add
- indices:admin/cache/clear
- indices:admin/close
- indices:admin/create
- indices:admin/data_stream/create
- indices:admin/data_stream/delete
- indices:admin/data_stream/get
- indices:admin/delete
- indices:admin/exists
- indices:admin/flush
- indices:admin/flush\*
- indices:admin/forcemerge
- indices:admin/get
- indices:admin/index_template/delete
- indices:admin/index_template/get
- indices:admin/index_template/put
- indices:admin/index_template/simulate
- indices:admin/index_template/simulate_index
- indices:admin/mapping/auto_put
- indices:admin/mapping/put
- indices:admin/mappings/fields/get
- indices:admin/mappings/fields/get\*
- indices:admin/mappings/get
- indices:admin/open
- indices:admin/refresh
- indices:admin/refresh\*
- indices:admin/refresh_search_analyzers
- indices:admin/reload
- indices:admin/resize
- indices:admin/resolve/index
- indices:admin/rollover
- indices:admin/seq_no/global_checkpoint_sync
- indices:admin/settings/update
- indices:admin/shards/search_shards
- indices:admin/shrink
- indices:admin/synced_flush
- indices:admin/template/delete
- indices:admin/template/get
- indices:admin/template/put
- indices:admin/types/exists
- indices:admin/upgrade
- indices:admin/validate/query

### 索引数据读取权限

- indices:data/read/async_search
- indices:data/read/explain
- indices:data/read/field_caps
- indices:data/read/field_caps\*
- indices:data/read/get
- indices:data/read/mget
- indices:data/read/mget\*
- indices:data/read/msearch
- indices:data/read/msearch/template
- indices:data/read/mtv
- indices:data/read/mtv\*
- indices:data/read/point_in_time/create
- indices:data/read/point_in_time/delete
- indices:data/read/point_in_time/readall
- indices:data/read/rank_eval
- indices:data/read/rollup/search
- indices:data/read/scroll
- indices:data/read/scroll/clear
- indices:data/read/search
- indices:data/read/search\*
- indices:data/read/search/template
- indices:data/read/tv

### 索引数据写入权限

- indices:data/write/bulk
- indices:data/write/bulk\*
- indices:data/write/delete
- indices:data/write/delete/byquery
- indices:data/write/index
- indices:data/write/reindex
- indices:data/write/update
- indices:data/write/update/byquery

### 索引监控权限

- indices:monitor/data_stream/stats
- indices:monitor/field_usage_stats
- indices:monitor/recovery
- indices:monitor/segments
- indices:monitor/settings/get
- indices:monitor/shard_stores
- indices:monitor/stats
- indices:monitor/upgrade

### ILM 索引级别权限

- indices:admin/ilm/managedindex

### 跨集群复制索引权限

- indices:admin/plugins/replication/autofollow/stats
- indices:admin/plugins/replication/follower/stats
- indices:admin/plugins/replication/index/all-shards
- indices:admin/plugins/replication/index/pause
- indices:admin/plugins/replication/index/resume
- indices:admin/plugins/replication/index/setup/validate
- indices:admin/plugins/replication/index/start
- indices:admin/plugins/replication/index/stats
- indices:admin/plugins/replication/index/status_check
- indices:admin/plugins/replication/index/stop
- indices:admin/plugins/replication/index/update
- indices:admin/plugins/replication/index/update_metadata
- indices:admin/plugins/replication/resources/release
- indices:admin/replication/index/all_status
- indices:data/read/plugins/replication/changes
- indices:data/read/plugins/replication/file_chunk
- indices:data/read/plugins/replication/file_metadata
- indices:data/write/plugins/replication/changes

## 权限集合

为了提高权限设置的效率，我们对系统权限进行自定义管理，从而快速批量选择一组相关的权限，而不是分别选择单个权限，为了方便，系统内置了若干权限集合。

### 超级权限

| 名称      | 描述                                      |
| :-------- | :---------------------------------------- |
| unlimited | 超级权限，集群和索引级别，等同于 `"*"` 。 |

### 集群级别

| 名称                            | 描述                                                                                          |
| :------------------------------ | :-------------------------------------------------------------------------------------------- |
| cluster_all                     | 授予全部集群级别的权限，等同于 `cluster:*`。                                                  |
| cluster_monitor                 | 授予全部集群的监控相关的权限，等同于 `cluster:monitor/*`。                                    |
| cluster_composite_ops_ro        | 授予请求的只读权限，如 `mget`、`msearch` 或 `mtv`，再加上别名和 `scroll` 的访问。             |
| cluster_composite_ops           | 与 `cluster_composite_ops_ro` 相同，但是额外包括 `bulk`、`reindex` 和所有的别名权限。         |
| manage_snapshots                | 授予所有和快照、仓库管理相关的权限，等同于 `cluster:admin/snapshot/*` 和 `cluster:admin/repository/*`。 |
| cluster_manage_index_templates  | 授予管理索引模板的权限，等同于 `indices:admin/template/*`。                                   |
| cluster_manage_pipelines        | 授予管理 Ingest Pipeline 的权限，等同于 `cluster:admin/ingest/pipeline/*`。                   |
| rollup                          | 授予所有 Rollup 相关的操作权限，包括创建、删除、启动、停止和搜索等。                         |
| manage_data_streams             | 授予管理数据流（Data Stream）的权限，等同于 `indices:admin/data_stream/*` 和 `indices:monitor/data_stream/stats`。 |

### 索引级别

| 名称            | 描述                                                                                 |
| :-------------- | :----------------------------------------------------------------------------------- |
| indices_all     | 授予全部索引级别的权限，等同于 `indices:*`。                                         |
| get             | 授予 `get` 和 `mget` 操作的权限。                                                    |
| read            | 授予只读相关的权限，如搜索、获取字段 Mapping、`get` 和 `mget` 操作。                 |
| write           | 授予在已有索引里面写入数据的权限，包括 `index`、`update`、`bulk` 和 `mapping/put`。   |
| index           | 授予索引文档的权限，包括 `index`、`update`、`bulk` 和 `mapping/put` 操作。            |
| delete          | 授予删除文档的权限。                                                                 |
| crud            | 增删改查的组合，等同于 `read` + `write`。                                            |
| search          | 授予文档搜索的权限，包括 `search`、`msearch`、`rollup/search` 和 `suggest` 权限。    |
| suggest         | 授予使用搜索提示 API 的权限。                                                        |
| data_access     | 授予所有数据读写权限，等同于 `indices:data/*` 加上 `crud`。                           |
| create_index    | 授予创建索引和 Mapping 的权限。                                                      |
| indices_monitor | 授予访问索引监控相关接口的权限（如：recovery、segments、index stats 和 status 等）。 |
| manage_aliases  | 授予管理别名的权限，等同于 `indices:admin/aliases*`。                                |
| manage          | 授予所有索引监控和管理相关的权限，等同于 `indices:monitor/*` 加 `indices:admin/*`。   |

## 内置角色

系统内置了以下角色，可以直接分配给用户：

| 角色名    | 描述                                                                                                   |
| :-------- | :----------------------------------------------------------------------------------------------------- |
| superuser | 超级用户角色，授予对所有集群 API 和所有索引的无限制访问权限。                                          |
| monitor   | 监控角色，授予集群健康和性能的监控权限，适用于运维和仪表盘用户。                                       |
| readonly  | 只读角色，授予对所有索引的只读访问权限，适用于数据消费者、分析人员和只读应用客户端。                   |

