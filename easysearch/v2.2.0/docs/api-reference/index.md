---
title: "API 手册"
date: 0001-01-01
description: "以 Easysearch API 为单位的说明，权限、参数、示例。"
summary: "API 手册 #  详细的 REST API 参考文档，包括权限、参数与调用示例。
🎯 常用 API 速查 #  # 集群健康 GET /_cluster/health # 节点统计 GET /_nodes/stats # 热线程 GET /_nodes/hot_threads # 索引大小 GET /_cat/indices?v&amp;h=index,store.size # 分片分布 GET /_cat/shards?v # 线程池 GET /_cat/thread_pool?v # JVM 统计 GET /_nodes/stats/jvm # 缓存统计 GET /_stats/query_cache  快速链接 #    常用 API  "
---


# API 手册

详细的 REST API 参考文档，包括权限、参数与调用示例。


## 🎯 常用 API 速查

```bash
# 集群健康
GET /_cluster/health

# 节点统计
GET /_nodes/stats

# 热线程
GET /_nodes/hot_threads

# 索引大小
GET /_cat/indices?v&h=index,store.size

# 分片分布
GET /_cat/shards?v

# 线程池
GET /_cat/thread_pool?v

# JVM 统计
GET /_nodes/stats/jvm

# 缓存统计
GET /_stats/query_cache
```

---


## 快速链接

- [常用 API]({{< relref "/docs/operations/cluster-admin/common" >}})
