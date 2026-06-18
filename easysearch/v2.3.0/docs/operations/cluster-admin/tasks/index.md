---
title: "任务管理"
date: 0001-01-01
description: "查看和管理集群任务。"
summary: "任务管理 #  任务是在集群中运行的任何操作。例如，搜索图书数据集以查找标题或作者姓名是一项任务。将自动创建任务以监视集群的运行状况和性能。有关集群中当前执行的所有任务的详细信息，可以使用 tasks API 操作。
以下请求返回有关所有任务的信息：
GET _tasks 通过包含任务 ID，您可以获得特定任务的信息。请注意，任务 ID 由节点的标识字符串和任务的数字 ID 组成。例如，如果节点的标识串是 nodestring ，任务的数字标识是 1234 ，则任务 ID 是 nodestring:1234 。您可以通过运行 tasks 操作来查找此信息。
GET _tasks/&lt;task_id&gt; 请注意，如果任务完成运行，它将不会作为请求的一部分返回。对于一个需要稍长时间才能完成的任务的示例，可以在较大的文档上运行 _reindex API 操作，然后运行 tasks 。
响应示例
{ &#34;nodes&#34;: { &#34;Mgqdm0f9SEGClWxp_RdnaQ&#34;: { &#34;name&#34;: &#34;easy-node1&#34;, &#34;transport_address&#34;: &#34;30.18.0.3:9300&#34;, &#34;host&#34;: &#34;30.18.0.3&#34;, &#34;ip&#34;: &#34;30.18.0.3:9300&#34;, &#34;roles&#34;: [&#34;data&#34;, &#34;ingest&#34;, &#34;master&#34;, &#34;remote_cluster_client&#34;], &#34;tasks&#34;: { &#34;Mgqdm0f9SEGClWxp_RdnaQ:17416&#34;: { &#34;node&#34;: &#34;Mgqdm0f9SEGClWxp_RdnaQ&#34;, &#34;id&#34;: 17416, &#34;type&#34;: &#34;transport&#34;, &#34;action&#34;: &#34;cluster:monitor/tasks/lists&#34;, &#34;start_time_in_millis&#34;: 1613599752458, &#34;running_time_in_nanos&#34;: 994000, &#34;cancellable&#34;: false, &#34;headers&#34;: {} }, &#34;Mgqdm0f9SEGClWxp_RdnaQ:17413&#34;: { &#34;node&#34;: &#34;Mgqdm0f9SEGClWxp_RdnaQ&#34;, &#34;id&#34;: 17413, &#34;type&#34;: &#34;transport&#34;, &#34;action&#34;: &#34;indices:data/write/bulk&#34;, &#34;start_time_in_millis&#34;: 1613599752286, &#34;running_time_in_nanos&#34;: 30846500, &#34;cancellable&#34;: false, &#34;parent_task_id&#34;: &#34;Mgqdm0f9SEGClWxp_RdnaQ:17366&#34;, &#34;headers&#34;: {} }, &#34;Mgqdm0f9SEGClWxp_RdnaQ:17366&#34;: { &#34;node&#34;: &#34;Mgqdm0f9SEGClWxp_RdnaQ&#34;, &#34;id&#34;: 17366, &#34;type&#34;: &#34;transport&#34;, &#34;action&#34;: &#34;indices:data/write/reindex&#34;, &#34;start_time_in_millis&#34;: 1613599750929, &#34;running_time_in_nanos&#34;: 1529733100, &#34;cancellable&#34;: true, &#34;headers&#34;: {} } } } } } 您还可以在查询中使用以下参数。"
---


# 任务管理

任务是在集群中运行的任何操作。例如，搜索图书数据集以查找标题或作者姓名是一项任务。将自动创建任务以监视集群的运行状况和性能。有关集群中当前执行的所有任务的详细信息，可以使用 `tasks` API 操作。

以下请求返回有关所有任务的信息：

```
GET _tasks
```

通过包含任务 ID，您可以获得特定任务的信息。请注意，任务 ID 由节点的标识字符串和任务的数字 ID 组成。例如，如果节点的标识串是 `nodestring` ，任务的数字标识是 `1234` ，则任务 ID 是 `nodestring:1234` 。您可以通过运行 `tasks` 操作来查找此信息。

```
GET _tasks/<task_id>
```

请注意，如果任务完成运行，它将不会作为请求的一部分返回。对于一个需要稍长时间才能完成的任务的示例，可以在较大的文档上运行 [`_reindex`](../data-management/reindex.md) API 操作，然后运行 `tasks` 。

**响应示例**

```json
{
  "nodes": {
    "Mgqdm0f9SEGClWxp_RdnaQ": {
      "name": "easy-node1",
      "transport_address": "30.18.0.3:9300",
      "host": "30.18.0.3",
      "ip": "30.18.0.3:9300",
      "roles": ["data", "ingest", "master", "remote_cluster_client"],
      "tasks": {
        "Mgqdm0f9SEGClWxp_RdnaQ:17416": {
          "node": "Mgqdm0f9SEGClWxp_RdnaQ",
          "id": 17416,
          "type": "transport",
          "action": "cluster:monitor/tasks/lists",
          "start_time_in_millis": 1613599752458,
          "running_time_in_nanos": 994000,
          "cancellable": false,
          "headers": {}
        },
        "Mgqdm0f9SEGClWxp_RdnaQ:17413": {
          "node": "Mgqdm0f9SEGClWxp_RdnaQ",
          "id": 17413,
          "type": "transport",
          "action": "indices:data/write/bulk",
          "start_time_in_millis": 1613599752286,
          "running_time_in_nanos": 30846500,
          "cancellable": false,
          "parent_task_id": "Mgqdm0f9SEGClWxp_RdnaQ:17366",
          "headers": {}
        },
        "Mgqdm0f9SEGClWxp_RdnaQ:17366": {
          "node": "Mgqdm0f9SEGClWxp_RdnaQ",
          "id": 17366,
          "type": "transport",
          "action": "indices:data/write/reindex",
          "start_time_in_millis": 1613599750929,
          "running_time_in_nanos": 1529733100,
          "cancellable": true,
          "headers": {}
        }
      }
    }
  }
}
```

您还可以在查询中使用以下参数。

| 参数                | 数据类型 | 描述                                                                                                                                                               |
| :------------------ | :------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| nodes               | List     | 以逗号分隔的节点 ID 或名称列表，用于限制返回的信息。使用 `_local` 从要连接的节点返回信息，指定节点名称以从特定节点获取信息，或将参数保持为空以从所有节点获取信息。 |
| actions             | List     | 应返回的操作的逗号分隔列表。保留为空以返回全部。                                                                                                                   |
| detailed            | Boolean  | 返回详细的任务信息。（Default: false）                                                                                                                             |
| parent_task_id      | String   | 返回具有指定父任务 ID（node_id:task_number）的任务。保持为空或设置为 -1 以返回全部。                                                                               |
| wait_for_completion | Boolean  | 等待匹配的任务完成。（Default: false）                                                                                                                             |
| group_by            | Enum     | 按父/子关系或节点对任务进行分组。（Default: nodes）                                                                                                                |
| timeout             | Time     | 显式操作超时。（Default: 30 seconds）                                                                                                                              |
| master_timeout      | Time     | 等待连接到主节点的时间。（Default: 30 seconds）                                                                                                                    |

例如，此请求返回当前在名为 `easy-node1` 的节点上运行的任务。

**请求示例**

```
GET /_tasks?nodes=easy-node1
```

**响应示例**

```json
{
  "nodes": {
    "Mgqdm0f9SEGClWxp_RdnaQ": {
      "name": "easy-node1",
      "transport_address": "sample_address",
      "host": "sample_host",
      "ip": "sample_ip",
      "roles": ["data", "ingest", "master", "remote_cluster_client"],
      "tasks": {
        "Mgqdm0f9SEGClWxp_RdnaQ:24578": {
          "node": "Mgqdm0f9SEGClWxp_RdnaQ",
          "id": 24578,
          "type": "transport",
          "action": "cluster:monitor/tasks/lists",
          "start_time_in_millis": 1611612517044,
          "running_time_in_nanos": 638700,
          "cancellable": false,
          "headers": {}
        },
        "Mgqdm0f9SEGClWxp_RdnaQ:24579": {
          "node": "Mgqdm0f9SEGClWxp_RdnaQ",
          "id": 24579,
          "type": "direct",
          "action": "cluster:monitor/tasks/lists[n]",
          "start_time_in_millis": 1611612517044,
          "running_time_in_nanos": 222200,
          "cancellable": false,
          "parent_task_id": "Mgqdm0f9SEGClWxp_RdnaQ:24578",
          "headers": {}
        }
      }
    }
  }
}
```

## 放弃任务

获取任务列表后，您可以使用以下请求取消所有可取消的任务：

```
POST _tasks/_cancel
```

请注意，并非所有任务都可以取消。要查看任务是否可取消，请参阅对 `tasks` API 请求的响应中的 `cancellable` 字段。

您还可以通过包含特定任务 ID 来取消任务。

```
POST _tasks/<task_id>/_cancel
```

`cancel` 操作支持与 `tasks` 操作相同的参数。以下示例显示如何取消多个节点上的所有可取消任务。

```
POST _tasks/_cancel?nodes=easy-node1,easy-node2
```

## 将头信息附加到任务

为了将请求与任务关联起来以便更好地跟踪，可以在 `curl` 命令的 HTTPS 请求读取器中提供 `X-Opaque-Id:<Id_number>` header 。API 将在返回的结果中附加指定的头。

用法：

```bash
curl -i -H "X-Opaque-Id: 111111" "https://localhost:9200/_tasks" -u 'admin:xxxxxxxxxxxx' --insecure
```

`_tasks` 操作返回以下结果。

```json
HTTP/1.1 200 OK
X-Opaque-Id: 111111
content-type: application/json; charset=UTF-8
content-length: 768

{
  "nodes": {
    "Mgqdm0f9SEGClWxp_RdnaQ": {
      "name": "easy-node1",
      "transport_address": "30.18.0.4:9300",
      "host": "30.18.0.4",
      "ip": "30.18.0.4:9300",
      "roles": [
        "data",
        "ingest",
        "master",
        "remote_cluster_client"
      ],
      "tasks": {
        "Mgqdm0f9SEGClWxp_RdnaQ:30072": {
          "node": "Mgqdm0f9SEGClWxp_RdnaQ",
          "id": 30072,
          "type": "direct",
          "action": "cluster:monitor/tasks/lists[n]",
          "start_time_in_millis": 161316670305,
          "running_time_in_nanos": 245400,
          "cancellable": false,
          "parent_task_id": "Mgqdm0f9SEGClWxp_RdnaQ:30071",
          "headers": {
            "X-Opaque-Id": "111111"
          }
        },
        "Mgqdm0f9SEGClWxp_RdnaQ:30071": {
          "node": "Mgqdm0f9SEGClWxp_RdnaQ",
          "id": 30071,
          "type": "transport",
          "action": "cluster:monitor/tasks/lists",
          "start_time_in_millis": 161316670305,
          "running_time_in_nanos": 658200,
          "cancellable": false,
          "headers": {
            "X-Opaque-Id": "111111"
          }
        }
      }
    }
  }
}
```

此操作支持与 `任务` 操作相同的参数。以下示例显示了如何将 `X-Opaque-Id` 与特定任务相关联。

```bash
curl -i -H "X-Opaque-Id: 123456" "https://localhost:9200/_tasks?nodes=easy-node1" -u 'admin:xxxxxxxxxxxx' --insecure
```

