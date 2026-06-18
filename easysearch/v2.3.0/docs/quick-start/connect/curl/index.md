---
title: "使用 Curl 访问 Easysearch"
date: 0001-01-01
description: "通过 curl 命令行与 Easysearch 进行 RESTful 交互：索引文档、检索、搜索、聚合。"
summary: "使用 Curl 访问 Easysearch #  Easysearch 提供 RESTful API，任何能发送 HTTP 请求的工具都可以与它交互。curl 是最常用的命令行工具，非常适合快速验证和学习。
请求格式 #  一个典型的 Easysearch 请求由以下部分组成：
curl -X&lt;VERB&gt; &#39;&lt;PROTOCOL&gt;://&lt;HOST&gt;:&lt;PORT&gt;/&lt;PATH&gt;?&lt;QUERY_STRING&gt;&#39; \  -H &#39;Content-Type: application/json&#39; \  -d &#39;&lt;BODY&gt;&#39;    部件 说明     VERB HTTP 方法：GET、POST、PUT、DELETE、HEAD   PROTOCOL http 或 https（Easysearch 默认启用 HTTPS）   HOST 节点主机名，本地为 localhost   PORT HTTP 服务端口，默认 9200   PATH API 路径，例如 _search、_count、my-index/_doc/1   QUERY_STRING 可选参数，例如 ?"
---


# 使用 Curl 访问 Easysearch

Easysearch 提供 RESTful API，任何能发送 HTTP 请求的工具都可以与它交互。`curl` 是最常用的命令行工具，非常适合快速验证和学习。

## 请求格式

一个典型的 Easysearch 请求由以下部分组成：

```bash
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' \
     -H 'Content-Type: application/json' \
     -d '<BODY>'
```

| 部件 | 说明 |
|------|------|
| `VERB` | HTTP 方法：`GET`、`POST`、`PUT`、`DELETE`、`HEAD` |
| `PROTOCOL` | `http` 或 `https`（Easysearch 默认启用 HTTPS） |
| `HOST` | 节点主机名，本地为 `localhost` |
| `PORT` | HTTP 服务端口，默认 `9200` |
| `PATH` | API 路径，例如 `_search`、`_count`、`my-index/_doc/1` |
| `QUERY_STRING` | 可选参数，例如 `?pretty` 格式化输出 |
| `BODY` | JSON 格式的请求体（如果需要） |

> 💡 **Easysearch 默认启用 HTTPS 和基础认证**，所以请求时通常需要加上 `-k`（跳过证书验证）和 `-u user:password` 参数。

## 验证集群连通性

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/?pretty'
```

返回示例：

```json
{
  "name" : "node-1",
  "cluster_name" : "my-cluster",
  "cluster_uuid" : "xxx",
  "version" : {
    "distribution" : "easysearch",
    "number" : "1.x.x",
    ...
  },
  "tagline" : "You Know, for Search"
}
```

查看集群健康状态：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/_cluster/health?pretty'
```

统计集群中的文档数量：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/_count?pretty' -H 'Content-Type: application/json' -d '
{
    "query": {
        "match_all": {}
    }
}'
```

## 索引文档

Easysearch 中，存储数据的行为叫做 **索引**（indexing）。向索引 `megacorp` 写入一个文档：

```bash
curl -k -u admin:admin -XPUT 'https://localhost:9200/megacorp/_doc/1?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "first_name" : "John",
    "last_name"  : "Smith",
    "age"        : 25,
    "about"      : "I love to go rock climbing",
    "interests"  : [ "sports", "music" ]
}'
```

路径 `/megacorp/_doc/1` 表示：索引 `megacorp`，文档 ID 为 `1`。

继续写入更多数据：

```bash
curl -k -u admin:admin -XPUT 'https://localhost:9200/megacorp/_doc/2?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "first_name" : "Jane",
    "last_name"  : "Smith",
    "age"        : 32,
    "about"      : "I like to collect rock albums",
    "interests"  : [ "music" ]
}'

curl -k -u admin:admin -XPUT 'https://localhost:9200/megacorp/_doc/3?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "first_name" : "Douglas",
    "last_name"  : "Fir",
    "age"        : 35,
    "about"      : "I like to build cabinets",
    "interests"  : [ "forestry" ]
}'
```

> 无需预先创建索引或定义字段类型，Easysearch 会自动完成。当然，生产环境建议使用显式 Mapping。

## 检索文档

通过 `GET` 请求 + 文档 ID 检索单个文档：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_doc/1?pretty'
```

返回结果包含文档元数据和 `_source` 字段（原始 JSON）：

```json
{
  "_index" : "megacorp",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "first_name" : "John",
    "last_name" : "Smith",
    "age" : 25,
    "about" : "I love to go rock climbing",
    "interests" : [ "sports", "music" ]
  }
}
```

> 💡 **HTTP 方法速记**：`PUT` 创建/覆盖文档，`GET` 检索文档，`DELETE` 删除文档，`HEAD` 检查文档是否存在。

## 搜索

### 搜索所有文档

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?pretty'
```

### 查询字符串搜索

搜索姓氏为 Smith 的员工：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?q=last_name:Smith&pretty'
```

### 使用 Query DSL

使用 JSON 请求体构造更复杂的查询——这是 Easysearch 推荐的搜索方式：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}'
```

### 组合查询：全文 + 过滤

搜索姓 Smith 且年龄大于 30 的员工：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "query" : {
        "bool" : {
            "must" : {
                "match" : { "last_name" : "Smith" }
            },
            "filter" : {
                "range" : { "age" : { "gt" : 30 } }
            }
        }
    }
}'
```

### 全文搜索

搜索 `about` 字段中包含 "rock climbing" 的文档——Easysearch 会按**相关性得分**排序：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "query" : {
        "match" : { "about" : "rock climbing" }
    }
}'
```

### 短语搜索

如果要精确匹配整个短语 "rock climbing"，使用 `match_phrase`：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "query" : {
        "match_phrase" : { "about" : "rock climbing" }
    }
}'
```

### 高亮搜索

在结果中高亮匹配的文本片段：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "query" : {
        "match_phrase" : { "about" : "rock climbing" }
    },
    "highlight" : {
        "fields" : { "about" : {} }
    }
}'
```

返回结果中会多出 `highlight` 字段，匹配文本被 `<em>` 标签包裹。

## 聚合分析

聚合（Aggregations）类似 SQL 的 `GROUP BY`，可以在搜索的同时做统计分析。

统计员工最受欢迎的兴趣爱好：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "size" : 0,
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests.keyword" }
        }
    }
}'
```

在查询结果基础上聚合——只统计姓 Smith 的员工：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "query" : { "match" : { "last_name" : "Smith" } },
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests.keyword" }
        }
    }
}'
```

嵌套聚合——按兴趣分组并计算每组的平均年龄：

```bash
curl -k -u admin:admin -XGET 'https://localhost:9200/megacorp/_search?pretty' \
     -H 'Content-Type: application/json' -d '
{
    "size" : 0,
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests.keyword" },
            "aggs" : {
                "avg_age" : { "avg" : { "field" : "age" } }
            }
        }
    }
}'
```

## 删除与清理

删除单个文档：

```bash
curl -k -u admin:admin -XDELETE 'https://localhost:9200/megacorp/_doc/1?pretty'
```

删除整个索引：

```bash
curl -k -u admin:admin -XDELETE 'https://localhost:9200/megacorp?pretty'
```

## 常用技巧

| 技巧 | 命令 |
|------|------|
| 格式化输出 | 加 `?pretty` 参数 |
| 查看 HTTP 头 | `curl -i ...` |
| 跳过 TLS 证书验证 | `curl -k ...` |
| 指定认证 | `curl -u admin:admin ...` |
| 查看所有索引 | `curl -k -u admin:admin 'https://localhost:9200/_cat/indices?v'` |
| 查看节点信息 | `curl -k -u admin:admin 'https://localhost:9200/_cat/nodes?v'` |
| 查看分片状态 | `curl -k -u admin:admin 'https://localhost:9200/_cat/shards?v'` |

## 下一步

- [Java 客户端]({{< relref "./java-client.md" >}})
- [Python 客户端]({{< relref "./python-client.md" >}})
- [入门教程]({{< relref "../tutorial.md" >}})：更完整的上手流程

