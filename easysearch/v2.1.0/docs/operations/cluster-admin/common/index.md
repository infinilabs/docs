---
title: "常用操作"
date: 0001-01-01
description: "集群与索引的高频管理操作示例。"
summary: "其它常用 API #  此页面包含 Easysearch 常用 API 的示例请求。
使用非默认设置创建索引 #  PUT my-logs { &#34;settings&#34;: { &#34;number_of_shards&#34;: 4, &#34;number_of_replicas&#34;: 2 }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;year&#34;: { &#34;type&#34;: &#34;integer&#34; } } } } 索引单个文档并自动生成随机 ID #  POST my-logs/_doc { &#34;title&#34;: &#34;Your Name&#34;, &#34;year&#34;: &#34;2016&#34; } 索引单个文档并指定 ID #  PUT my-logs/_doc/1 { &#34;title&#34;: &#34;Weathering with You&#34;, &#34;year&#34;: &#34;2019&#34; } 一次索引多个文档 #  请求正文末尾的空白行是必填的。如果省略 _id 字段， Easysearch 将生成一个随机 id 。"
---


# 其它常用 API

此页面包含 Easysearch 常用 API 的示例请求。

## 使用非默认设置创建索引

```json
PUT my-logs
{
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 2
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "year": {
        "type": "integer"
      }
    }
  }
}
```

## 索引单个文档并自动生成随机 ID

```json
POST my-logs/_doc
{
  "title": "Your Name",
  "year": "2016"
}
```

## 索引单个文档并指定 ID

```json
PUT my-logs/_doc/1
{
  "title": "Weathering with You",
  "year": "2019"
}
```

## 一次索引多个文档

请求正文末尾的空白行是必填的。如果省略 `_id` 字段， Easysearch 将生成一个随机 id 。

```json
POST _bulk
{ "index": { "_index": "my-logs", "_id": "2" } }
{ "title": "The Garden of Words", "year": 2013 }
{ "index" : { "_index": "my-logs", "_id" : "3" } }
{ "title": "5 Centimeters Per Second", "year": 2007 }

```

## 列出所有索引

```
GET _cat/indices?v
```

## 打开或关闭与模式匹配的所有索引

```
POST my-logs*/_open
POST my-logs*/_close
```

## 删除与模式匹配的所有索引

```
DELETE my-logs*
```

## 创建索引别名

此请求为索引 `my-logs-2019-11-13` 创建别名 `my-logs-today` 。

```
PUT my-logs-2019-11-13/_alias/my-logs-today
```

## 列出所有别名

```
GET _cat/aliases?v
```

## 搜索单个索引或与模式匹配的所有索引

```
GET my-logs/_search?q=test
GET my-logs*/_search?q=test
```

## 获取群集设置，包括默认值

```
GET _cluster/settings?include_defaults=true
```

## 更改磁盘使用限制

```json
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.disk.watermark.low": "80%",
    "cluster.routing.allocation.disk.watermark.high": "85%"
  }
}
```

## 获取群集运行状况

```
GET _cluster/health
```

## 列出群集中的节点

```
GET _cat/nodes?v
```

## 获取节点统计信息

```
GET _nodes/stats
```

## 在存储库中获取快照

```
GET _snapshot/my-repository/_all
```

## 生成快照

```
PUT _snapshot/my-repository/my-snapshot
```

## 从快照还原

```json
POST _snapshot/my-repository/my-snapshot/_restore
{
  "indices": "-.security",
  "include_global_state": false
}
```
## 统计索引每个字段的访问次数

```
GET metrics/_field_usage_stats
```

## 分析指定索引每个字段的磁盘占用大小

```
POST metrics/_disk_usage?run_expensive_tasks=true
```
