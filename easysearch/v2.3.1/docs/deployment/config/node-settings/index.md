---
title: "节点配置"
date: 0001-01-01
description: "easysearch.yml 静态配置项详细参考。"
summary: "节点配置 #  本章详细介绍 easysearch.yml 中每个节点需要配置的静态设置。这些设置在修改后需要重启对应节点才能生效。
 集群级别的动态配置（可通过 _cluster/settings API 在线修改的配置）请参见 集群配置。
 配置子文档 #     文档 涵盖内容      集群与节点 cluster.name、node.name、node.roles、node.attr.*    路径配置 path.data、path.logs、path.repo    网络配置 network.host、http.port、transport.port、HTTP 调优、CORS    集群发现 discovery.seed_hosts、cluster.initial_master_nodes、故障检测    网关与恢复 gateway.recover_after_*、分片恢复行为    内存与缓存 bootstrap.memory_lock、fielddata / query / request 缓存    安全配置 security.* TLS、审计、DN、REST API、系统索引保护    JVM 配置 jvm."
---


# 节点配置

本章详细介绍 `easysearch.yml` 中每个节点需要配置的**静态设置**。这些设置在修改后需要重启对应节点才能生效。

> 集群级别的动态配置（可通过 `_cluster/settings` API 在线修改的配置）请参见 [集群配置]({{< relref "../configuration.md" >}})。

## 配置子文档

| 文档 | 涵盖内容 |
|------|----------|
| [集群与节点]({{< relref "./cluster-node.md" >}}) | `cluster.name`、`node.name`、`node.roles`、`node.attr.*` |
| [路径配置]({{< relref "./path.md" >}}) | `path.data`、`path.logs`、`path.repo` |
| [网络配置]({{< relref "./network.md" >}}) | `network.host`、`http.port`、`transport.port`、HTTP 调优、CORS |
| [集群发现]({{< relref "./discovery.md" >}}) | `discovery.seed_hosts`、`cluster.initial_master_nodes`、故障检测 |
| [网关与恢复]({{< relref "./gateway.md" >}}) | `gateway.recover_after_*`、分片恢复行为 |
| [内存与缓存]({{< relref "./memory.md" >}}) | `bootstrap.memory_lock`、fielddata / query / request 缓存 |
| [安全配置]({{< relref "./security-settings.md" >}}) | `security.*` TLS、审计、DN、REST API、系统索引保护 |
| [JVM 配置]({{< relref "./jvm.md" >}}) | `jvm.options` 堆内存、GC、OOM dump |
| [日志配置]({{< relref "./logging.md" >}}) | `log4j2.properties`、日志文件、慢日志 |
| [线程池与断路器]({{< relref "./thread-pool.md" >}}) | `thread_pool.*`、`indices.breaker.*`、API 兼容 |

## 配置文件一览

Easysearch 的配置文件位于安装目录下的 `config/` 目录（可通过环境变量 `ES_PATH_CONF` 自定义）。

| 文件 | 用途 |
|------|------|
| `easysearch.yml` | 节点与集群配置的主文件 |
| `jvm.options` | JVM 启动参数（堆大小、GC 等） |
| `log4j2.properties` | 日志配置（级别、格式、输出位置） |
| `config/security/*.yml` | 安全模块本地配置（用户、角色等） |

### 指定配置目录

```bash
ES_PATH_CONF=/path/to/my/config ./bin/easysearch
```

> 使用 systemd、Docker 或自带脚本运行服务时，需在对应的服务配置中设置 `ES_PATH_CONF`。

## 配置文件格式（YAML）

`easysearch.yml` 使用 YAML 格式。几个常用规则：

### 层级写法

```yaml
path:
  data: /var/lib/easysearch
  logs: /var/log/easysearch
```

等价的扁平 key 写法：

```yaml
path.data: /var/lib/easysearch
path.logs: /var/log/easysearch
```

### 列表写法

多行形式：

```yaml
discovery.seed_hosts:
  - 192.168.1.10:9300
  - 192.168.1.11
  - seeds.mydomain.com
```

单行数组形式：

```yaml
discovery.seed_hosts: ["192.168.1.10:9300", "192.168.1.11", "seeds.mydomain.com"]
```

### 环境变量替换

在 `easysearch.yml` 中可以使用 `${...}` 引用环境变量：

```yaml
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

环境变量的值会被当作纯字符串读取。如果需要列表效果，可以用逗号分隔：

```bash
export SEED_HOSTS="10.0.0.1:9300,10.0.0.2:9300"
```

```yaml
discovery.seed_hosts: ${SEED_HOSTS}
```

## 配置原则

- `easysearch.yml` 只放**节点本地 / 集群引导**配置，保持所有节点上内容简单且一致。
- 集群级别调优参数通过 [集群配置 API]({{< relref "../configuration.md" >}}) 以 `persistent` 方式修改。
- **切勿将未受保护的节点暴露在公共互联网上！**
