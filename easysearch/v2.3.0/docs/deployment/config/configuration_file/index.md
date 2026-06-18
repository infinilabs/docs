---
title: "配置说明"
date: 0001-01-01
summary: "配置文件 #  可以在每个 Easysearch 节点上找到 easysearch.yml ，通常位于 Easysearch 安装目录下 config/easysearch.yml。
配置文件一览 #  Easysearch 主要有以下几类配置文件：
 easysearch.yml：节点与集群配置的主文件，包括网络、发现、角色、路径等。  只放“本机相关”和“集群引导”类配置（如 cluster.name、node.name、network.host、discovery.seed_hosts 等）更易维护。   jvm.options：JVM 启动参数，如堆大小（-Xms/-Xmx）、GC 设置等。 log4j2.properties：日志配置，控制日志级别、输出格式与输出位置。 config/security/*.yml：安全模块本地配置，用于内置用户、角色等，详见 安全配置。  配置目录位置 #   默认情况下，配置目录位于 $ES_HOME/config。 你可以通过环境变量 ES_PATH_CONF 指定一个自定义配置目录，例如：  ES_PATH_CONF=/path/to/my/config ./bin/easysearch 在使用 systemd、Docker 或自带脚本运行服务时，通常需要在对应的服务配置里设置 ES_PATH_CONF，而不是只在当前 shell 里导出。
配置文件格式要点（YAML） #  Easysearch 的配置文件使用 YAML 格式，几个常用规则：
 缩进代表层级：  path: data: /var/lib/easysearch logs: /var/log/easysearch  同样的内容也可以写成“扁平 key”：  path.data: /var/lib/easysearch path."
---


# 配置文件

可以在每个 Easysearch 节点上找到 `easysearch.yml` ，通常位于 Easysearch 安装目录下 `config/easysearch.yml`。

## 配置文件一览

Easysearch 主要有以下几类配置文件：

- `easysearch.yml`：**节点与集群配置的主文件**，包括网络、发现、角色、路径等。  
  - 只放“本机相关”和“集群引导”类配置（如 `cluster.name`、`node.name`、`network.host`、`discovery.seed_hosts` 等）更易维护。
- `jvm.options`：**JVM 启动参数**，如堆大小（`-Xms`/`-Xmx`）、GC 设置等。
- `log4j2.properties`：**日志配置**，控制日志级别、输出格式与输出位置。
- `config/security/*.yml`：**安全模块本地配置**，用于内置用户、角色等，详见[安全配置]({{< relref "/docs/operations/security/_index.md" >}})。

### 配置目录位置

- 默认情况下，配置目录位于 `$ES_HOME/config`。
- 你可以通过环境变量 `ES_PATH_CONF` 指定一个自定义配置目录，例如：

```bash
ES_PATH_CONF=/path/to/my/config ./bin/easysearch
```

在使用 systemd、Docker 或自带脚本运行服务时，通常需要在对应的服务配置里设置 `ES_PATH_CONF`，而不是只在当前 shell 里导出。

## 配置文件格式要点（YAML）

Easysearch 的配置文件使用 YAML 格式，几个常用规则：

- **缩进代表层级**：

```yaml
path:
  data: /var/lib/easysearch
  logs: /var/log/easysearch
```

- 同样的内容也可以写成“扁平 key”：

```yaml
path.data: /var/lib/easysearch
path.logs: /var/log/easysearch
```

- **列表（sequence）** 可以写成多行：

```yaml
discovery.seed_hosts:
  - 192.168.1.10:9300
  - 192.168.1.11
  - seeds.mydomain.com
```

也可以写成单行数组形式：

```yaml
discovery.seed_hosts: ["192.168.1.10:9300", "192.168.1.11", "seeds.mydomain.com"]
```

## 环境变量替换

在 `easysearch.yml` 中，可以使用 `${...}` 的形式引用环境变量，Easysearch 启动时会自动替换。例如：

```yaml
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

注意事项：

- 环境变量的值会被当作**纯字符串**读取。
- 如果需要“列表”效果，可以使用逗号分隔的字符串，然后由 Easysearch 解析为列表，例如：

```bash
export SEED_HOSTS="10.0.0.1:9300,10.0.0.2:9300"
```

```yaml
discovery.seed_hosts: ${SEED_HOSTS}
```

## 配置作用域与推荐用法

- **静态设置**：只能在 `easysearch.yml` 或启动参数里配置，例如路径、网络、节点角色等，修改后需要重启节点。
- **动态设置**：可以通过 `_cluster/settings` 接口在运行时修改，例如多数集群级别的调优项。

推荐实践：

- `easysearch.yml`：只放**节点本地 / 集群引导**配置，保持所有节点上内容简单且一致。
- 集群级别调优：使用[集群配置](./configuration.md) 中介绍的 **集群设置 API** 来做持久（`persistent`）或临时（`transient`）修改。

### 警告

**切勿将未受保护的节点暴露在公共互联网上！**

## JVM 配置

详见 [JVM 选项]({{< relref "./node-settings/jvm" >}})，包括堆内存、垃圾回收器等。

## 日志配置

详见 [日志配置]({{< relref "./node-settings/logging" >}})，包括日志位置、级别调整、慢日志设置。

## 安全配置

`config/security/` 目录下的配置文件说明详见 [本地配置（YAML）]({{< relref "/docs/operations/security/configuration/yaml" >}})

## Elasticsearch 兼容性

Easysearch 支持与 Elasticsearch 8.x API 的兼容性模式：

```yaml
elasticsearch.api_compatibility: true
elasticsearch.api_compatibility_version: "8.9.0"
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `elasticsearch.api_compatibility` | `false` | 是否启用 Elasticsearch 兼容性模式。启用后，Easysearch 将支持部分 Elasticsearch 8.x API 接口 |
| `elasticsearch.api_compatibility_version` | 无 | 指定兼容的 Elasticsearch 版本。例如 `"8.9.0"` 表示兼容到 Elasticsearch 8.9.0 |

**使用场景**：
- 从 Elasticsearch 迁移到 Easysearch 时，保持现有应用无需修改
- 需要逐步迁移的环境

---

## 常用网络设置

> 完整的 `easysearch.yml` 配置参考（包括索引、分片分配、断路器、安全、线程池等全部配置项）请参见 [集群配置]({{< relref "./configuration.md" >}})。

Easysearch 默认只绑定到 localhost。对于生产环境的集群，需要配置基本的网络设置：

| 参数名称                | 属性 | 功能说明                                                                            | 默认值                      | 生产环境示例                 |
|------------------------|------|------------------------------------------------------------------------------------|-----------------------------|---------------------------|
| network.host           | 静态 | 节点绑定的主机名或 IP 地址。                                                          | `_local_`                   | `192.168.1.10`            |
| network.bind_host      | 静态 | 节点监听传入请求的具体地址。可以配置为外网地址（例如 0.0.0.0 监听所有接口）或其他特定的地址。   | 随 `network.host`           | `0.0.0.0`                 |
| network.publish_host   | 静态 | 节点间通信的通告地址。在多网卡或多网络环境中，应显式设置 network.publish_host。             | 随 `network.host`           | `192.168.1.10`            |
| http.port              | 静态 | HTTP 请求的绑定端口。支持单个值或范围。指定范围时，节点将绑定到该范围内的第一个可用端口。       | `9200-9300`                 | `9200`                    |
| transport.port         | 静态 | 节点间通信 (TCP) 端口。支持单个值或范围。指定范围时，节点将绑定到该范围内第一个可用端口。在所有具有主节点资格（master-eligible）的节点上，请将此项设置为单个值。   | `9300-9400`                      | `9300`                    |
| discovery.seed_hosts   | 静态 | 提供集群中有资格成为主节点的节点地址列表。也可以是一个包含多个以逗号分隔的地址的字符串。每个地址的格式为 host:port 或 host。端口按顺序检查以下设置来确定： transport.profiles.default.port，transport.port，未设置时默认9300。       | `["127.0.0.1", "[::1]"]`。       | `["10.0.0.1", "10.0.0.2"]` |
| discovery.type         | 静态 | 指定 Easysearch 是否应形成一个多节点集群。如果将 discovery.type 设置为 single-node，Easysearch 将形成一个单节点集群。                                  | `zen` (默认形成多节点集群，无需设置)  | `single-node`             |
| cluster.initial_master_nodes | 静态 | 设置新集群中初始的主节点候选节点列表。默认情况下，此列表为空，意味着该节点期望加入已经引导好的集群。在生产环境中首次启动新的 Easysearch 集群时，必须配置 cluster.initial_master_nodes，以明确哪些节点有资格参与主节点的选举。 当集群完成了首次主节点选举，集群已经正常运行时，就不再需要此设置了。这时应该从每个节点的配置中移除这项设置。 | `[]` 默认为空                 | `["node-1", "node-2"]`    |
| node.roles             | 静态 | 定义节点的角色。节点默认具有右侧列出的角色。如果设置了 node.roles，则节点只会被分配指定的角色。                                                           | `[master, data, ingest, remote_cluster_client]` | `["data", "ingest"]`      |

### 分布式集群模式

以下配置适用于 Easysearch 的分布式集群模式，确保各节点可以发现彼此并组成一个集群，分布式模式建议 3 个独立的 Master 节点，配置如下：

```
cluster.name: easysearch
node.roles: [ "master" ]
network.host: 0.0.0.0
http.port: 19201
transport.port: 19301
discovery.seed_hosts: ["192.168.101.5:19301", "192.168.101.6:19302", "192.168.101.7:19303"]
cluster.initial_master_nodes: ["192.168.101.5:19301", "192.168.101.6:19302", "192.168.101.7:19303"]
```

数据节点配置如下：

```
cluster.name: easysearch
node.roles: [ "data" ]
network.host: 0.0.0.0
http.port: 19200
transport.port: 19300
discovery.seed_hosts: ["192.168.101.5:19301", "192.168.101.6:19302", "192.168.101.7:19303"]
```

> Master 使用 3 个就够了，数据节点可以根据动态增加。


### 单节点服务模式

如果希望单个 Easysearch 节点模拟集群，能监听所有网卡并提供对外服务供其他主机访问，也就是单节点模式，可以使用以下配置：

```
cluster.name: easysearch
network.host: 0.0.0.0
http.port: 13200
transport.port: 13300
cluster.initial_master_nodes: ["localhost:13300"]
```


了解更多，请查看[搭建集群]({{< relref "/docs/operations/cluster-admin/cluster" >}})

