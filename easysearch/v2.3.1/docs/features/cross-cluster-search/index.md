---
title: "跨集群搜索"
date: 0001-01-01
description: "一次查询覆盖多个集群，无需数据迁移即可实现全局统一搜索与分析。"
summary: "跨集群搜索（CCS） #  Easysearch 跨集群搜索（Cross-Cluster Search, CCS）允许用户在本地集群发起一个查询请求，同时检索分布在不同地理位置、不同业务环境中的多个远程集群——无需进行数据迁移或归集，即可实现全局视角下的即时分析。
 核心能力 #  统一视图，全局洞察 #  一个查询请求即可覆盖全公司范围内的索引数据，无需切换访问点，实现跨业务线的综合分析。
降低成本，按需存储 #  数据保留在产生的源集群中，避免大规模数据传输带来的网络带宽损耗与存储冗余成本。
系统解耦与灵活性 #  各个集群可以独立升级、独立维护，通过 CCS 实现逻辑上的互联互通，降低超大规模架构的运维复杂度。
故障隔离 #  若某个远程集群暂时不可用，CCS 可配置为忽略故障集群并返回其余可用部分的搜索结果，保障服务不中断。
 快速上手 #  1. 配置远程集群连接 #  在本地集群上注册远程集群连接。远程集群连接支持 Sniff 和 Proxy 两种访问模式。
Sniff 模式是默认访问模式，本地集群会连接配置的远程种子节点，并通过远程集群返回的节点信息发现更多可连接节点。配置时填写远程集群 transport 地址（默认端口 9300）：
PUT _cluster/settings { &#34;persistent&#34;: { &#34;cluster.remote&#34;: { &#34;cluster-beijing&#34;: { &#34;seeds&#34;: [&#34;beijing-node:9300&#34;] }, &#34;cluster-shanghai&#34;: { &#34;seeds&#34;: [&#34;shanghai-node:9300&#34;] } } } } 如果本地集群不能直接访问远程集群的所有 transport 节点，只能通过固定代理地址或网关地址访问远程集群，可以使用 Proxy 模式。此时 proxy_address 应填写代理或网关的监听地址："
---


# 跨集群搜索（CCS）

Easysearch 跨集群搜索（Cross-Cluster Search, CCS）允许用户在本地集群发起一个查询请求，同时检索分布在不同地理位置、不同业务环境中的多个远程集群——无需进行数据迁移或归集，即可实现全局视角下的即时分析。

---

## 核心能力

### 统一视图，全局洞察

一个查询请求即可覆盖全公司范围内的索引数据，无需切换访问点，实现跨业务线的综合分析。

### 降低成本，按需存储

数据保留在产生的源集群中，避免大规模数据传输带来的网络带宽损耗与存储冗余成本。

### 系统解耦与灵活性

各个集群可以独立升级、独立维护，通过 CCS 实现逻辑上的互联互通，降低超大规模架构的运维复杂度。

### 故障隔离

若某个远程集群暂时不可用，CCS 可配置为忽略故障集群并返回其余可用部分的搜索结果，保障服务不中断。

---

## 快速上手

### 1. 配置远程集群连接

在本地集群上注册远程集群连接。远程集群连接支持 Sniff 和 Proxy 两种访问模式。

Sniff 模式是默认访问模式，本地集群会连接配置的远程种子节点，并通过远程集群返回的节点信息发现更多可连接节点。配置时填写远程集群 transport 地址（默认端口 9300）：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.remote": {
      "cluster-beijing": {
        "seeds": ["beijing-node:9300"]
      },
      "cluster-shanghai": {
        "seeds": ["shanghai-node:9300"]
      }
    }
  }
}
```

如果本地集群不能直接访问远程集群的所有 transport 节点，只能通过固定代理地址或网关地址访问远程集群，可以使用 Proxy 模式。此时 `proxy_address` 应填写代理或网关的监听地址：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.remote": {
      "cluster-beijing": {
        "mode": "proxy",
        "proxy_address": "beijing-proxy.example.com:19300",
        "seeds": null
      }
    }
  }
}
```

`proxy_address` 使用代理入口地址，端口是代理监听端口，不是 HTTP 端口 9200。代理后端应能转发到远程集群的 transport 端口。

例如本地测试中启动 `127.0.0.1:19300 -> 127.0.0.1:9300` 的 TCP 代理时，应填写代理监听地址 `127.0.0.1:19300`。

`proxy_address` 只有在 `mode` 为 `proxy` 时有效。如果同一个 alias 之前使用 Sniff 模式配置过 `seeds`，切换到 Proxy 模式时建议同时将 `cluster.remote.<alias>.seeds` 置为 `null`，避免旧配置残留。

Proxy 模式只改变本地集群连接远程集群的方式，不改变跨集群搜索语法。连接成功后，仍然使用 `集群名:索引名` 的格式查询远程索引；remote-only 查询、本地和远程混合查询、远程通配符索引、`_msearch` 都使用相同的写法。

### 2. 跨集群搜索

使用 `集群名:索引名` 的格式指定远程索引：

```json
POST /cluster-beijing:logs-*,cluster-shanghai:logs-*/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1d/d",
        "lte": "now"
      }
    }
  },
  "size": 20
}
```

### 3. 同时搜索本地和远程索引

```json
POST /local-logs-*,cluster-beijing:logs-*/_search
{
  "query": {
    "match": { "message": "error" }
  },
  "size": 10
}
```

### 4. 跨集群聚合

```json
POST /cluster-beijing:metrics-*,cluster-shanghai:metrics-*/_search
{
  "size": 0,
  "aggs": {
    "region_stats": {
      "terms": { "field": "region.keyword" },
      "aggs": {
        "avg_latency": {
          "avg": { "field": "latency" }
        }
      }
    }
  }
}
```

---

## 连接管理

### 查看远程集群状态

```json
GET _remote/info
```

返回所有已配置的远程集群的连接状态、连接模式，以及 Sniff 模式的种子节点或 Proxy 模式的代理地址。

### 动态更新连接

支持实时添加或移除远程集群，无需重启：

添加 Sniff 模式远程集群：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.remote.cluster-guangzhou.seeds": ["guangzhou-node:9300"]
  }
}
```

添加 Proxy 模式远程集群：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.remote.cluster-guangzhou.mode": "proxy",
    "cluster.remote.cluster-guangzhou.proxy_address": "guangzhou-proxy.example.com:19300",
    "cluster.remote.cluster-guangzhou.seeds": null
  }
}
```

### 移除远程集群

移除 Sniff 模式远程集群：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.remote.cluster-guangzhou.seeds": null
  }
}
```

移除 Proxy 模式远程集群：

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.remote.cluster-guangzhou.seeds": null,
    "cluster.remote.cluster-guangzhou.mode": null,
    "cluster.remote.cluster-guangzhou.proxy_address": null,
    "cluster.remote.cluster-guangzhou.proxy_socket_connections": null,
    "cluster.remote.cluster-guangzhou.server_name": null,
    "cluster.remote.cluster-guangzhou.skip_unavailable": null
  }
}
```

---

## 安全与权限

- 支持基于安全凭证的远程访问
- 跨集群通信链路加密（TLS）
- 严格遵循各集群原有的数据访问权限
- 非管理员用户需要映射到适当的角色权限

---

## 应用场景

| 场景 | 说明 |
|------|------|
| **多地域业务统一搜索** | 各区域数据本地存储、本地写入，总部通过 CCS 统一查询 |
| **两地三中心容灾架构** | 双活或多活数据中心下，数据就近读取与异地备份查询 |
| **历史数据归档查询** | 历史数据保存在独立集群中，按需查询，避免热集群负担 |
| **平滑架构演进** | 在集群拆分、迁移、升级过程中，保持业务查询连续性 |

---

## 相关文档

- [跨集群复制]({{< relref "/docs/operations/cluster-admin/ccr" >}})
- [集群扩容与分片]({{< relref "/docs/operations/cluster-admin/cluster" >}})

