---
title: "集群配置"
date: 0001-01-01
description: "节点与集群核心配置项——生产环境必须关注的设置。"
summary: "配置 #  Easysearch 默认配置已针对多数场景优化。本文聚焦生产环境必须关注的配置项。
 原则：如果不确定是否需要改，就不要改。过度调优往往适得其反。
  配置速查表 #     类别 关键配置 修改方式 风险等级     标识 cluster.name、node.name 静态（重启生效） 低   路径 path.data、path.logs 静态（重启生效） 高   网络 network.host、http.port、transport.port 静态（重启生效） 中   发现 discovery.seed_hosts、cluster.initial_master_nodes 静态（重启生效） 高   内存 ES_HEAP_SIZE、jvm.options 静态（重启生效） 高   恢复 gateway.recover_after_* 静态（重启生效） 中   集群级 大部分 cluster.*、indices.* 动态 API 视具体配置     必须配置 #  集群和节点名称 #  # easysearch."
---


# 配置

Easysearch 默认配置已针对多数场景优化。本文聚焦生产环境**必须关注**的配置项。

> **原则**：如果不确定是否需要改，就不要改。过度调优往往适得其反。

---

## 配置速查表

| 类别 | 关键配置 | 修改方式 | 风险等级 |
|------|----------|----------|----------|
| **标识** | `cluster.name`、`node.name` | 静态（重启生效） | 低 |
| **路径** | `path.data`、`path.logs` | 静态（重启生效） | 高 |
| **网络** | `network.host`、`http.port`、`transport.port` | 静态（重启生效） | 中 |
| **发现** | `discovery.seed_hosts`、`cluster.initial_master_nodes` | 静态（重启生效） | 高 |
| **内存** | `ES_HEAP_SIZE`、`jvm.options` | 静态（重启生效） | 高 |
| **恢复** | `gateway.recover_after_*` | 静态（重启生效） | 中 |
| **集群级** | 大部分 `cluster.*`、`indices.*` | 动态 API | 视具体配置 |

---

## 必须配置

### 集群和节点名称

```yaml
# easysearch.yml
cluster.name: my-cluster-prod    # 避免节点误加入其他集群
node.name: node-01               # 便于日志排查和监控识别
```

> 不设置 `node.name` 时，每次重启会随机生成——凌晨 3 点排查问题时，你不会想面对一堆随机名称。

### 数据目录

```yaml
# easysearch.yml
path.data: /data/easysearch      # 独立于安装目录
path.logs: /var/log/easysearch   # 便于日志收集
```

**为什么重要**：默认数据存放在安装目录下，升级或重装时可能被误删。

### 网络绑定

```yaml
# easysearch.yml
network.host: 10.0.0.1           # 内网 IP
http.port: 9200
transport.port: 9300
```

**安全建议**：
- 生产环境**不要**绑定 `0.0.0.0` 或公网 IP
- 通过 Nginx/Gateway 代理对外暴露

### 集群发现

```yaml
# easysearch.yml
discovery.seed_hosts:
  - node-01.internal:9300
  - node-02.internal:9300
  - node-03.internal:9300

# 仅首次启动集群时需要
cluster.initial_master_nodes:
  - node-01
  - node-02
  - node-03
```

**关键点**：
- 列出所有 Master 候选节点
- `cluster.initial_master_nodes` 在集群建立后可以删除
- **永远不要使用组播**——生产环境只用单播

---

## 内存配置

### JVM 堆设置

```bash
# 方式 1：环境变量
export ES_HEAP_SIZE=16g

# 方式 2：jvm.options
-Xms16g
-Xmx16g
```

**三条铁律**：

| 规则 | 说明 |
|------|------|
| `Xms` = `Xmx` | 避免运行时调整堆大小 |
| 堆 ≤ 物理内存的 50% | 另一半留给 Lucene 文件缓存 |
| 堆 ≤ 31 GB | 超过 32GB 会禁用指针压缩，得不偿失 |

**典型配置**：

| 物理内存 | JVM 堆 | 说明 |
|----------|--------|------|
| 16 GB | 8 GB | 开发/测试 |
| 64 GB | 31 GB | 生产推荐 |
| 128 GB | 31 GB | 堆不要超过 31GB |

### 禁用 Swap

内存交换到磁盘会让性能下降 100 倍以上。

```bash
# 临时禁用
sudo swapoff -a

# 或降低 swappiness
echo "vm.swappiness = 1" >> /etc/sysctl.conf
sysctl -p
```

也可以锁定 JVM 内存：

```yaml
# easysearch.yml
bootstrap.memory_lock: true
```

验证是否生效：

```json
GET _nodes?filter_path=**.mlockall
// 返回 "mlockall": true 表示成功
```

---

## 集群恢复配置

集群重启时，避免在所有节点上线前触发不必要的分片迁移。

```yaml
# easysearch.yml
gateway.recover_after_nodes: 8      # 至少 8 个节点在线才开始恢复
gateway.expected_nodes: 10          # 集群预期节点数
gateway.recover_after_time: 5m      # 最多等待 5 分钟
```

**行为**：
1. 等待至少 8 个节点上线
2. 如果 5 分钟内 10 个节点都上线了，立即开始恢复
3. 如果 5 分钟后只有 8-9 个节点，也开始恢复

**效果**：大集群重启时，分片恢复从数小时缩短到几秒钟。

---

## 动态配置

运行时可通过 API 调整的配置，无需重启。

### 查看当前配置

```json
GET _cluster/settings?include_defaults=true
```

### 修改配置

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "all"
  }
}
```

**配置优先级**（从高到低）：
1. Transient（临时，重启后失效）
2. Persistent（持久，写入集群状态）
3. easysearch.yml
4. 默认值

### 常用动态配置

| 配置 | 用途 | 示例值 |
|------|------|--------|
| `cluster.routing.allocation.enable` | 控制分片分配 | `all` / `none` |
| `cluster.routing.rebalance.enable` | 控制分片再平衡 | `all` / `none` |
| `indices.recovery.max_bytes_per_sec` | 恢复带宽限制 | `100mb` |
| `cluster.routing.allocation.disk.watermark.low` | 磁盘低水位 | `85%` |
| `cluster.routing.allocation.disk.watermark.high` | 磁盘高水位 | `90%` |

---

## 日志配置

日志通过 `config/log4j2.properties` 配置。

### 调整日志级别

```properties
# 临时调试时调高级别
logger.action.level = debug

# 问题解决后恢复
logger.action.level = info
```

### 慢查询日志

```json
PUT /my-index/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s"
}
```

**最佳实践**：
- 默认保持 INFO 级别
- 使用 Filebeat 收集日志到集中平台
- 不要在节点本地保留过多历史日志

---

## 配置验证

### 节点启动后检查

```json
# 检查节点信息
GET _nodes?filter_path=nodes.*.name,nodes.*.roles,nodes.*.jvm.mem

# 检查关键设置
GET _nodes/settings?filter_path=nodes.*.settings.path,nodes.*.settings.network
```

### 常见问题排查

| 问题 | 可能原因 | 检查命令 |
|------|----------|----------|
| 节点无法加入集群 | 发现配置错误 | 检查 `discovery.seed_hosts` |
| 内存不足 | 堆设置不当 | `GET _nodes/stats/jvm` |
| 分片不分配 | 磁盘水位超限 | `GET _cluster/allocation/explain` |
| 启动慢 | 恢复配置缺失 | 检查 `gateway.*` 配置 |

---

## 小结

| 配置类别 | 必须设置 | 推荐值 |
|----------|----------|--------|
| 集群名称 | ✓ | 有意义的名称 |
| 节点名称 | ✓ | 有意义的名称 |
| 数据路径 | ✓ | 独立于安装目录 |
| 网络绑定 | ✓ | 内网 IP |
| 发现配置 | ✓ | 显式 seed 列表 |
| JVM 堆 | ✓ | 50% 内存，≤ 31GB |
| 禁用 Swap | ✓ | `swapoff -a` 或 `memory_lock` |

**相关文档**：
- [节点配置]({{< relref "/docs/deployment/config/node-settings/" >}})：完整参数说明
- [集群配置]({{< relref "/docs/deployment/config/configuration.md" >}})：动态配置详解
- [容量规划]({{< relref "/docs/best-practices/index-design.md" >}})：硬件选型指南
- [监控]({{< relref "./monitoring" >}})：配置监控告警



