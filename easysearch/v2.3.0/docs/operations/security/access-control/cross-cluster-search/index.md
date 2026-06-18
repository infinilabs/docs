---
title: "跨集群搜索权限"
date: 0001-01-01
summary: "跨集群搜索权限 #  跨集群搜索允许集群中的节点对其他集群执行搜索请求。Easysearch 原生支持跨集群搜索，本页重点说明权限与配置要点。
相关指南（先读这些） #    权限控制总览  分布式搜索基础  安全与多租户最佳实践  身份验证流程 #  当跨集群搜索通过 协调集群 访问 远程集群 时：
 安全模块对协调集群上的用户进行身份验证。 安全模块在协调集群上获取用户的后端角色。 请求调用（包括经过身份验证的用户）将转发到远程集群。 在远程群集上评估用户的权限。  远程群集和协调集群可以分别配置不同的身份验证和授权配置，但我们建议在两者上使用相同的设置。
权限信息 #  要查询远程集群上的索引，除了 READ 或 SEARCH 权限外，用户还需要具有以下索引权限：
indices:admin/shards/search_shards role.yml 样例配置 #  humanresources: cluster: - CLUSTER_COMPOSITE_OPS_RO indices: &#34;humanresources&#34;: &#34;*&#34;: - READ - indices:admin/shards/search_shards # needed for CCS 配置流程 #  分别启动两个集群，如下：
➜ curl -k &#39;https://localhost:9200/_cluster/health?pretty&#39; -u admin:xxxxxxxxxxxx { &#34;cluster_name&#34; : &#34;easysearch&#34;, &#34;status&#34; : &#34;green&#34;, &#34;timed_out&#34; : false, &#34;number_of_nodes&#34; : 1, &#34;number_of_data_nodes&#34; : 1, &#34;active_primary_shards&#34; : 1, &#34;active_shards&#34; : 1, &#34;relocating_shards&#34; : 0, &#34;initializing_shards&#34; : 0, &#34;unassigned_shards&#34; : 0, &#34;delayed_unassigned_shards&#34; : 0, &#34;number_of_pending_tasks&#34; : 0, &#34;number_of_in_flight_fetch&#34; : 0, &#34;task_max_waiting_in_queue_millis&#34; : 0, &#34;active_shards_percent_as_number&#34; : 100."
---


# 跨集群搜索权限

跨集群搜索允许集群中的节点对其他集群执行搜索请求。Easysearch 原生支持跨集群搜索，本页重点说明权限与配置要点。

## 相关指南（先读这些）

- [权限控制总览]({{< relref "/docs/operations/security/access-control/_index.md" >}})
- [分布式搜索基础]({{< relref "/docs/fundamentals/distributed-search.md" >}})
- [安全与多租户最佳实践]({{< relref "/docs/best-practices/security-and-multi-tenancy.md" >}})

## 身份验证流程

当跨集群搜索通过 _协调集群_ 访问 _远程集群_ 时：

- 安全模块对协调集群上的用户进行身份验证。
- 安全模块在协调集群上获取用户的后端角色。
- 请求调用（包括经过身份验证的用户）将转发到远程集群。
- 在远程群集上评估用户的权限。

远程群集和协调集群可以分别配置不同的身份验证和授权配置，但我们建议在两者上使用相同的设置。

## 权限信息

要查询远程集群上的索引，除了 `READ` 或 `SEARCH` 权限外，用户还需要具有以下索引权限：

```
indices:admin/shards/search_shards
```

#### role.yml 样例配置

```yml
humanresources:
  cluster:
    - CLUSTER_COMPOSITE_OPS_RO
  indices:
    "humanresources":
      "*":
        - READ
        - indices:admin/shards/search_shards # needed for CCS
```

## 配置流程

分别启动两个集群，如下：

```bash
➜  curl -k  'https://localhost:9200/_cluster/health?pretty' -u admin:xxxxxxxxxxxx
{
  "cluster_name" : "easysearch",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
➜  curl -k  'https://localhost:9201/_cluster/health?pretty' -u admin:xxxxxxxxxxxx
{
  "cluster_name" : "my-application22",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

在协调群集上，添加远程群集名称和 IP 地址（端口为 9300）：

```json
curl -k -XPUT -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx' 'https://localhost:9200/_cluster/settings' -d '
{
  "persistent": {
    "cluster.remote": {
      "cluster1": {
        "seeds": ["127.0.0.1:9300"]
      },
      "cluster2": {
          "seeds": ["127.0.0.1:9301"]
        }
    }
  }
}'
```

在远程集群内索引一个文档：

```bash
curl -XPUT -k -H 'Content-Type: application/json' -u 'admin:xxxxxxxxxxxx' 'https://localhost:9201/books/_doc/1' -d '{"Dracula": "Bram Stoker"}'
```

此时，跨集群搜索已经可以正常工作。您可以使用 `admin` 用户进行测试：

```bash
✗ curl -XGET -k -u 'admin:xxxxxxxxxxxx' 'https://localhost:9200/cluster2:books/_search?pretty'
{
  "took" : 57,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "_clusters" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "cluster2:books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "Dracula" : "Bram Stoker"
        }
      }
    ]
  }
}
```

继续测试，在两个集群上分别创建一个新用户：

```bash
curl -XPUT -k -u 'admin:xxxxxxxxxxxx' 'https://localhost:9200/_security/user/booksuser' -H 'Content-Type: application/json' -d '{"password":"password"}'
curl -XPUT -k -u 'admin:xxxxxxxxxxxx' 'https://localhost:9201/_security/user/booksuser' -H 'Content-Type: application/json' -d '{"password":"password"}'
```

Then run the same search as before with `booksuser` :

```json
curl -XGET -k -u booksuser:password 'https://localhost:9200/cluster2:books/_search?pretty'
{
  "error" : {
    "root_cause" : [
      {
        "type" : "security_exception",
        "reason" : "no permissions for [indices:admin/shards/search_shards, indices:data/read/search] and User [name=booksuser, roles=[], requestedTenant=null]"
      }
    ],
    "type" : "security_exception",
    "reason" : "no permissions for [indices:admin/shards/search_shards, indices:data/read/search] and User [name=booksuser, roles=[], requestedTenant=null]"
  },
  "status" : 403
}
```

请注意权限错误。在远程群集上，创建具有适当权限的角色，并将 `booksuser` 映射到该角色：

```bash
curl -XPUT -k -u 'admin:xxxxxxxxxxxx' -H 'Content-Type: application/json' 'https://localhost:9200/_security/role/booksrole' -d '{"indices":[{"names":["books"],"privileges":["indices:admin/shards/search_shards","indices:data/read/search"]}]}'
curl -XPUT -k -u 'admin:xxxxxxxxxxxx' -H 'Content-Type: application/json' 'https://localhost:9200/_security/role_mapping/booksrole' -d '{"users" : ["booksuser"]}'

curl -XPUT -k -u 'admin:xxxxxxxxxxxx' -H 'Content-Type: application/json' 'https://localhost:9201/_security/role/booksrole' -d '{"indices":[{"names":["books"],"privileges":["indices:admin/shards/search_shards","indices:data/read/search"]}]}'
curl -XPUT -k -u 'admin:xxxxxxxxxxxx' -H 'Content-Type: application/json' 'https://localhost:9201/_security/role_mapping/booksrole' -d '{"users" : ["booksuser"]}'

```

两个集群都必须具有该用户，但只有远程集群需要角色和映射；在这种情况下，协调群集处理身份验证（即 “此请求是否包含有效的用户凭据？”），远程群集处理授权（即 “此用户是否可以访问此数据？”）。

重新搜索一次：

```bash
curl -XGET -k -u booksuser:password 'https://localhost:9200/cluster2:books/_search?pretty'
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "_clusters" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "cluster2:books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "Dracula" : "Bram Stoker"
        }
      }
    ]
  }
}
```

