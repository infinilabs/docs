---
title: "跨集群复制"
date: 0001-01-01
description: "跨集群数据同步、容灾备份、读写分离、多活架构的完整指南和 API 参考。"
summary: "跨集群复制（CCR） #  Easysearch 跨集群复制（Cross-Cluster Replication, CCR）支持将数据从源集群（Leader）实时同步到一个或多个目标集群（Follower）。
 使用跨集群复制 API 管理跨集群复制。
在跨集群复制中，可以将数据索引到一个领导者索引，然后 Easysearch 将这些数据复制到一个或多个只读的跟随者索引。所有在领导者上进行的后续操作都会在跟随者上复制，例如创建、更新或删除文档。
先决条件 #   1.11.1 版本之前，leader 和 follower 集群都必须安装 cross-cluster-replication 插件和 index-management 插件，1.11.1 版本开始，已经内置了 cross-cluster-replication 模块。 从1.15.2版本开始，cross-cluster-replication 和 index-management 都已经内置到 modules，不再需要安装。 如果 follower 集群的 easysearch.yml 文件中覆盖了 node.roles，确保它也包括 remote_cluster_client 角色，默认启用。 node.roles: [&lt;other_roles&gt;, remote_cluster_client]  权限 #   确保安全功能在两个集群上都启用或都禁用。如果启用了安全功能，确保非管理员用户被映射到适当的权限，以便他们可以执行复制操作。  部署示例集群 #   在本地起 2 个单节点的 easysearch 测试集群，分别是 follower-application (9201 端口) 和 leader-application (9200 端口) 在 easysearch."
---


# 跨集群复制（CCR）

Easysearch 跨集群复制（Cross-Cluster Replication, CCR）支持将数据从源集群（Leader）实时同步到一个或多个目标集群（Follower）。

---

使用跨集群复制 API 管理跨集群复制。

在跨集群复制中，可以将数据索引到一个领导者索引，然后 Easysearch 将这些数据复制到一个或多个只读的跟随者索引。所有在领导者上进行的后续操作都会在跟随者上复制，例如创建、更新或删除文档。

## 先决条件

- 1.11.1 版本之前，leader 和 follower 集群都必须安装 cross-cluster-replication 插件和 index-management 插件，1.11.1 版本开始，已经内置了 cross-cluster-replication 模块。
- 从1.15.2版本开始，cross-cluster-replication 和 index-management 都已经内置到 modules，不再需要安装。
- 如果 follower 集群的 easysearch.yml 文件中覆盖了 node.roles，确保它也包括 remote_cluster_client 角色，默认启用。
  `node.roles: [<other_roles>, remote_cluster_client]`

### 权限

- 确保安全功能在两个集群上都启用或都禁用。如果启用了安全功能，确保非管理员用户被映射到适当的权限，以便他们可以执行复制操作。

### 部署示例集群

- 在本地起 2 个单节点的 easysearch 测试集群，分别是 follower-application (9201 端口) 和 leader-application (9200 端口)
- 在 easysearch.yml 添加 discovery.type: single-node
- 如果启用 security 功能，确保 2 个集群的证书互信，测试环境可以直接合并 2 个节点的 ca 证书：
  例如 cat ca.crt ../../easysearch-1.0.0_2/config/ca.crt > trust-chain.pem
- 分别设置 security.ssl.transport.ca_file: trust-chain.pem

---

## 设置跨集群复制

### 设置跨群集连接

跨集群复制采用“pull”模型，在 follower 集群上，添加每个种子节点的 IP 地址（端口 9300），
为连接提供一个描述性名称，您将在请求中使用该名称来启动复制。以下是对应的 curl 命令：

```auto
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx' 'https://localhost:9201/_cluster/settings?pretty' -d '
{
  "persistent": {
    "cluster": {
      "remote": {
        "my-connection-alias": {
          "seeds": ["127.0.0.1:9300"]
        }
      }
    }
  }
}'
```

---

### 开始复制

首先，在 leader 集群创建 leader-01 索引, 并向索引写入数据。

```auto
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx' 'https://localhost:9200/leader-01?pretty'
```

然后设置跟随者集群开始复制。在请求体中，提供你想要复制的连接名和领导者索引，以及你想要使用的安全角色：

```auto
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx' 'https://localhost:9201/_replication/follower-01/_start?pretty' -d '
{
   "leader_alias": "my-connection-alias",
   "leader_index": "leader-01",
   "use_roles":{
      "leader_cluster_role": "superuser",
      "follower_cluster_role": "superuser"
   }
}'
```
因为演示用户是 admin 所以可以直接用 superuser 角色进行复制。

### 确认复制状态

```auto
curl -XGET -k -u 'admin:xxxxxxxxxxxx' 'https://localhost:9201/_replication/follower-01/_status?pretty'
```

响应

```json
{
  "status" : "SYNCING",
  "reason" : "User initiated",
  "leader_alias" : "my-connection-alias",
  "leader_index" : "leader-01",
  "follower_index" : "follower-01",
  "syncing_details" : {
    "leader_checkpoint" : 23,
    "follower_checkpoint" : 23,
    "seq_no" : 24
  }
}
```

### 列出 follow 集群所有正在复制的索引

包括同步中、暂停、失败等状态。
```auto
GET /_replication/all_status
```

响应

下面状态为 FAILED 或 PAUSED 的 索引 是因为 leader 集群启用了 ILM，将过期的索引删除了，CCR 会将被删除的索引在 status 中标记出来。

```json
[
  {
    "status": "FAILED",
    "reason": "Pause failed with \"Index log-test-000006 is already paused\". Original failure for initiating pause - [[log-test-000006][0] - org.easysearch.index.IndexNotFoundException - \"no such index [log-test-000006]\"], ",
    "leader_alias": "leader",
    "leader_index": "log-test-000006",
    "follower_index": "log-test-000006"
  },
  {
    "status": "SYNCING",
    "reason": "User initiated",
    "leader_alias": "leader",
    "leader_index": "test2",
    "follower_index": "test2",
    "syncing_details": {
      "leader_checkpoint": 12,
      "follower_checkpoint": 12,
      "seq_no": 13
    }
  },
  {
    "status": "PAUSED",
    "reason": "AutoPaused: [[log-test-000001][0] - org.easysearch.index.IndexNotFoundException - \"no such index [log-test-000001]\"], ",
    "leader_alias": "leader",
    "leader_index": "log-test-000001",
    "follower_index": "log-test-000001"
  },
  {
    "status": "PAUSED",
    "reason": "AutoPaused: [[log-test-000003][0] - org.easysearch.index.IndexNotFoundException - \"no such index [log-test-000003]\"], ",
    "leader_alias": "leader",
    "leader_index": "log-test-000003",
    "follower_index": "log-test-000003"
  },
  {
    "status": "SYNCING",
    "reason": "User initiated",
    "leader_alias": "leader",
    "leader_index": "test1",
    "follower_index": "test1",
    "syncing_details": {
      "leader_checkpoint": 20,
      "follower_checkpoint": 20,
      "seq_no": 21
    }
  },
  {
    "status": "PAUSED",
    "reason": "AutoPaused: [[log-test-000002][0] - org.easysearch.index.IndexNotFoundException - \"no such index [log-test-000002]\"], ",
    "leader_alias": "leader",
    "leader_index": "log-test-000002",
    "follower_index": "log-test-000002"
  },
  {
    "status": "FAILED",
    "reason": "Pause failed with \"Index log-test-000005 is already paused\". Original failure for initiating pause - [[log-test-000005][0] - org.easysearch.index.IndexNotFoundException - \"no such index [log-test-000005]\"], ",
    "leader_alias": "leader",
    "leader_index": "log-test-000005",
    "follower_index": "log-test-000005"
  },
  {
    "status": "PAUSED",
    "reason": "AutoPaused: [[log-test-000004][0] - org.easysearch.index.IndexNotFoundException - \"no such index [log-test-000004]\"], ",
    "leader_alias": "leader",
    "leader_index": "log-test-000004",
    "follower_index": "log-test-000004"
  },
  {
    "status": "SYNCING",
    "reason": "User initiated",
    "leader_alias": "leader",
    "leader_index": "log-test-000013",
    "follower_index": "log-test-000013",
    "syncing_details": {
      "leader_checkpoint": -1,
      "follower_checkpoint": -1,
      "seq_no": 0
    }
  },
  {
    "status": "SYNCING",
    "reason": "User initiated",
    "leader_alias": "leader",
    "leader_index": "log-test-000014",
    "follower_index": "log-test-000014",
    "syncing_details": {
      "leader_checkpoint": -1,
      "follower_checkpoint": -1,
      "seq_no": 0
    }
  }
]
```

### 暂停和恢复复制

如果您需要修复问题或减轻主集群的负载，您可以暂时暂停索引的复制：

```auto
curl -XPOST -k -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx' 'https://localhost:9201/_replication/follower-01/_pause?pretty' -d '{}'
```

恢复

```auto
curl -XPOST -ku 'admin:xxxxxxxxxxxx' -H 'Content-Type: application/json' 'https://localhost:9201/_replication/follower-01/_resume?pretty' -d '{}'
```

### 获取 leader 集群状态

在 leader 集群执行

```auto
GET /_replication/leader_stats
```

响应

```json
{
  "num_replicated_indices": 4,
  "operations_read": 0,
  "translog_size_bytes": 0,
  "operations_read_lucene": 0,
  "operations_read_translog": 0,
  "total_read_time_lucene_millis": 0,
  "total_read_time_translog_millis": 0,
  "bytes_read": 0,
  "index_stats": {
    "test1": {
      "operations_read": 0,
      "translog_size_bytes": 0,
      "operations_read_lucene": 0,
      "operations_read_translog": 0,
      "total_read_time_lucene_millis": 0,
      "total_read_time_translog_millis": 0,
      "bytes_read": 0
    },
    "test2": {
      "operations_read": 0,
      "translog_size_bytes": 0,
      "operations_read_lucene": 0,
      "operations_read_translog": 0,
      "total_read_time_lucene_millis": 0,
      "total_read_time_translog_millis": 0,
      "bytes_read": 0
    },
    "log-test-000013": {
      "operations_read": 0,
      "translog_size_bytes": 0,
      "operations_read_lucene": 0,
      "operations_read_translog": 0,
      "total_read_time_lucene_millis": 0,
      "total_read_time_translog_millis": 0,
      "bytes_read": 0
    },
    "log-test-000014": {
      "operations_read": 0,
      "translog_size_bytes": 0,
      "operations_read_lucene": 0,
      "operations_read_translog": 0,
      "total_read_time_lucene_millis": 0,
      "total_read_time_translog_millis": 0,
      "bytes_read": 0
    }
  }
}
```
### 获取 follower 集群状态为同步中的索引的信息

```auto
curl -XGET -ku 'admin:xxxxxxxxxxxx' 'https://localhost:9200/_replication/follower_stats'
```

响应

```json
{
  "num_syncing_indices": 4,
  "num_bootstrapping_indices": 0,
  "num_paused_indices": 5,
  "num_failed_indices": 2,
  "num_shard_tasks": 4,
  "num_index_tasks": 4,
  "operations_written": 0,
  "operations_read": 0,
  "failed_read_requests": 0,
  "throttled_read_requests": 0,
  "failed_write_requests": 0,
  "throttled_write_requests": 0,
  "follower_checkpoint": 30,
  "leader_checkpoint": 0,
  "total_write_time_millis": 0,
  "index_stats": {
    "test1": {
      "operations_written": 0,
      "operations_read": 0,
      "failed_read_requests": 0,
      "throttled_read_requests": 0,
      "failed_write_requests": 0,
      "throttled_write_requests": 0,
      "follower_checkpoint": 20,
      "leader_checkpoint": 0,
      "total_write_time_millis": 0
    },
    "test2": {
      "operations_written": 0,
      "operations_read": 0,
      "failed_read_requests": 0,
      "throttled_read_requests": 0,
      "failed_write_requests": 0,
      "throttled_write_requests": 0,
      "follower_checkpoint": 12,
      "leader_checkpoint": 0,
      "total_write_time_millis": 0
    },
    "log-test-000014": {
      "operations_written": 0,
      "operations_read": 0,
      "failed_read_requests": 0,
      "throttled_read_requests": 0,
      "failed_write_requests": 0,
      "throttled_write_requests": 0,
      "follower_checkpoint": -1,
      "leader_checkpoint": 0,
      "total_write_time_millis": 0
    },
    "log-test-000015": {
      "operations_written": 0,
      "operations_read": 0,
      "failed_read_requests": 0,
      "throttled_read_requests": 0,
      "failed_write_requests": 0,
      "throttled_write_requests": 0,
      "follower_checkpoint": -1,
      "leader_checkpoint": 0,
      "total_write_time_millis": 0
    }
  }
}
```

### 停止复制

在 follower 集群执行

```auto
curl -XPOST -k -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx' 'https://localhost:9201/_replication/follower-01/_stop?pretty' -d '{}'
```

## 跨集群复制的自动跟随

自动开始对匹配指定模式的索引进行复制。如果领导者集群上的新索引匹配该模式，Easysearch 会自动创建一个跟随者索引。
自动跟随让你可以根据匹配的模式自动复制在领导者集群上创建的索引。当你在领导者集群上创建一个名称与指定模式匹配的索引（例如，index-01\*），在跟随者集群上会自动创建一个对应的跟随者索引。

你可以为单个集群配置多个复制规则。目前的模式只支持通配符匹配。

在启用自动跟随之前，确认在两个集群之间设置了跨集群连接。

如果启用了安全插件，确保非管理员用户被映射到适当的权限，以便他们可以执行复制操作。关于索引和集群级别的权限要求，请参阅跨集群复制权限。

复制规则是你针对单个跟随者集群创建的模式集合。当你创建一个复制规则时，它首先会自动复制任何匹配模式的现有索引。然后，它将继续复制你创建的任何新索引，只要这些新索引匹配该模式。

示例 在 follower 集群执行：

```auto
curl -XPOST -k -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx' 'https://localhost:9201/_replication/_autofollow?pretty' -d '
{
   "leader_alias" : "my-connection-alias",
   "name": "my-replication-rule",
   "pattern": "nginx*",
   "use_roles":{
      "leader_cluster_role": "superuser",
      "follower_cluster_role": "superuser"
   }
}'

```

然后 创建 nginx 开头的索引并写入数据

### 查看 follower 集群的索引状态

```auto
curl -kuadmin:admin 'https://localhost:9201/_cat/indices?v'
```

如果发现文档数没变化可以执行 refresh 命令后再查看

### 查看已设置的自动跟随规则

```auto
curl -u 'admin:xxxxxxxxxxxx' -k 'https://localhost:9200/_replication/autofollow_stats'
```

### 删除自动跟随规则

可以通过在从集群上调用 API 来删除自动跟随。调用 API 仅停止任何新的自动跟随活动，并不会停止已由自动跟随启动的复制，要停止现有的复制活动并打开索引进行写入，请使用停止复制API操作。

示例

```auto
curl -XDELETE -k -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx' 'https://localhost:9201/_replication/_autofollow?pretty' -d '
{
   "leader_alias" : "my-connection-alias",
   "name": "my-replication-rule"
}'
```

---

## 如何使用自定义用户

如果不使用 admin 用户，必须将你自定义的用户绑定到
replication_leader，它在领导者集群上提供复制权限，
replication_follower，它在追随者集群上提供复制权限。

示例 分别在 leader 和 follower 集群执行创建 replication_user 用户的命令：

```auto
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx'  'https://xxxx:9xxx/_security/user/replication_user' -d '
{
  "password": "123456",
  "roles": ["replication_leader", "replication_follower"]
}'
```

### 用 replication_user 用户设置跟随者集群开始复制

```auto
curl -XPUT -k -H 'Content-Type: application/json' -u 'replication_user:123456' 'https://localhost:9288/_replication/follower-01/_start?pretty' -d '
{
   "leader_alias": "my-connection-alias",
   "leader_index": "leader-01",
   "use_roles":{
      "leader_cluster_role": "replication_leader",
      "follower_cluster_role": "replication_follower"
   }
}'
```

### 用 replication_user 用户设置跟随者集群自动复制

```auto
curl -XPOST -k -H 'Content-Type: application/json' -u 'replication_user:123456' 'https://localhost:9288/_replication/_autofollow?pretty' -d '
{
   "leader_alias" : "my-connection-alias",
   "name": "my-replication-rule",
   "pattern": "leader*",
   "use_roles":{
      "leader_cluster_role": "replication_leader",
      "follower_cluster_role": "replication_follower"
   }
}'
```

## 配置项

跨集群复制有多个配置项，这些配置项是可以动态修改的，可以将设置标记为 persistent 或 transient。

例如，要更新跟随集群轮询主集群以获取更新的频率：

```auto
PUT _cluster/settings
{
  "persistent": {
    "replication.follower.metadata_sync_interval": "30s"
  }
}
```
这些配置项管理远程恢复所消耗的资源。我们不建议更改这些设置；默认设置应适用于大多数使用场景。

| 设置                                                        | 默认值 | 描述                                                     |
|------------------------------------------------------------|--------|--------------------------------------------------------|
| replication.follower.index.recovery.chunk_size            | 10MB   | follower 集群为文件传输请求的块大小。指定块大小的值和单位,例如10MB、5KB。请参阅支持的单位。 |
| replication.follower.index.recovery.max_concurrent_file_chunks | 4      | 每个恢复过程中可以并行发送的文件块请求数。                                  |
| replication.follower.index.ops_batch_size           | 50000  | 在复制的同步阶段,一次可以获取的操作数量。                                  |
| replication.follower.concurrent_readers_per_shard   | 2      | 在复制的同步阶段,每个分片上追随者集群的并发请求数。                             |
| replication.autofollow.fetch_poll_interval          | 30s    | 自动跟随任务轮询leader集群以获取新的匹配索引的频率。                          |
| replication.follower.metadata_sync_interval         | 60s    | follower 集群轮询leader集群以获取更新的索引元数据的频率。                   |
| replication.translog.retention_lease.pruning.enabled | true   | 如果启用,基于保留租约来修剪translog。                                |
| replication.translog.retention_size                | 512MB  | 控制leader索引上translog的大小。                                |

[](#top)

