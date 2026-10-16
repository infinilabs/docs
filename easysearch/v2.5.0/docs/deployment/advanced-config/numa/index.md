---
title: "NUMA 配置"
date: 0001-01-01
summary: "NUMA 配置指南 #  在多路服务器（2 路及以上 CPU）上部署 Easysearch 时，NUMA（Non-Uniform Memory Access）拓扑会显著影响性能。不当的配置可能导致跨节点内存访问，增加延迟 30%–50%。
NUMA 基础概念 #  ┌──────────────┐ ┌──────────────┐ │ NUMA Node 0│ │ NUMA Node 1│ │ CPU 0-15 │ │ CPU 16-31 │ │ 本地内存 128G│◄──►│ 本地内存 128G│ │ 延迟 ~80ns │ QPI│ 延迟 ~80ns │ └──────────────┘ └──────────────┘ 本地访问快 跨节点访问慢 (~130ns)  每个 CPU 有自己的本地内存，访问本地内存最快 访问远端 NUMA 节点的内存需要通过互联总线（QPI/UPI），延迟更高 JVM 大堆 + 跨 NUMA 节点 = GC 停顿时间增加  查看 NUMA 拓扑 #  # 查看 NUMA 节点信息 numactl --hardware # 输出示例 available: 2 nodes (0-1) node 0 cpus: 0 1 2 3 4 5 6 7 16 17 18 19 20 21 22 23 node 0 size: 131072 MB node 1 cpus: 8 9 10 11 12 13 14 15 24 25 26 27 28 29 30 31 node 1 size: 131072 MB node distances: node 0 1 0: 10 21 1: 21 10 # 查看当前内存分配策略 numactl --show 推荐配置方案 #  方案一：绑定单个 NUMA 节点（推荐） #  将 Easysearch 进程绑定到一个 NUMA 节点，确保所有内存访问都在本地："
---


# NUMA 配置指南

在多路服务器（2 路及以上 CPU）上部署 Easysearch 时，NUMA（Non-Uniform Memory Access）拓扑会显著影响性能。不当的配置可能导致跨节点内存访问，增加延迟 30%–50%。

## NUMA 基础概念

```
┌──────────────┐    ┌──────────────┐
│   NUMA Node 0│    │   NUMA Node 1│
│  CPU 0-15    │    │  CPU 16-31   │
│  本地内存 128G│◄──►│  本地内存 128G│
│  延迟 ~80ns  │ QPI│  延迟 ~80ns  │
└──────────────┘    └──────────────┘
      本地访问快              跨节点访问慢 (~130ns)
```

- 每个 CPU 有自己的本地内存，访问本地内存最快
- 访问远端 NUMA 节点的内存需要通过互联总线（QPI/UPI），延迟更高
- JVM 大堆 + 跨 NUMA 节点 = GC 停顿时间增加

## 查看 NUMA 拓扑

```bash
# 查看 NUMA 节点信息
numactl --hardware

# 输出示例
available: 2 nodes (0-1)
node 0 cpus: 0 1 2 3 4 5 6 7 16 17 18 19 20 21 22 23
node 0 size: 131072 MB
node 1 cpus: 8 9 10 11 12 13 14 15 24 25 26 27 28 29 30 31
node 1 size: 131072 MB
node distances:
node   0   1
  0:  10  21
  1:  21  10
```

```bash
# 查看当前内存分配策略
numactl --show
```

## 推荐配置方案

### 方案一：绑定单个 NUMA 节点（推荐）

将 Easysearch 进程绑定到一个 NUMA 节点，确保所有内存访问都在本地：

```bash
# 绑定到 NUMA node 0
numactl --cpunodebind=0 --membind=0 /data/easysearch/bin/easysearch
```

适用场景：
- 单机部署一个 Easysearch 实例
- JVM 堆 ≤ 单个 NUMA 节点的内存（通常够用，31G 堆 + 页缓存）

### 方案二：每个 NUMA 节点一个实例

在大型服务器上，每个 NUMA 节点运行一个 Easysearch 实例：

```bash
# 实例 1：绑定 NUMA node 0，HTTP 端口 9200
numactl --cpunodebind=0 --membind=0 /data/easysearch-node1/bin/easysearch

# 实例 2：绑定 NUMA node 1，HTTP 端口 9201
numactl --cpunodebind=1 --membind=1 /data/easysearch-node2/bin/easysearch
```

每个实例配置独立的：
- `http.port`
- `transport.port`
- `path.data`
- `node.name`

### 方案三：交错分配（不推荐用于搜索场景）

```bash
# 内存在所有 NUMA 节点间交错分配
numactl --interleave=all /data/easysearch/bin/easysearch
```

交错分配平均化了延迟，但放弃了本地访问的优势，通常不如方案一。

## systemd 集成

如果使用 systemd 管理 Easysearch，在 service 文件中配置：

```ini
[Service]
ExecStart=/usr/bin/numactl --cpunodebind=0 --membind=0 /data/easysearch/bin/easysearch
```

## JVM 相关配置

配合 NUMA 绑定，JVM 参数建议：

```bash
# config/jvm.options
# 启用 NUMA 感知的内存分配（G1 GC）
-XX:+UseNUMA

# 大页内存（可选，需要 OS 配置）
-XX:+UseLargePages
```

### 配置大页内存（可选）

```bash
# 计算所需大页数（堆 31G / 大页 2MB = 15872 页，预留额外）
echo 16000 > /proc/sys/vm/nr_hugepages

# 持久化
echo "vm.nr_hugepages=16000" >> /etc/sysctl.conf
```

## 验证 NUMA 绑定

```bash
# 查看进程的 NUMA 分配
numastat -p $(pgrep -f easysearch | head -1)

# 查看内存是否集中在绑定的节点
# 期望：绑定节点的内存使用远高于其他节点
```

## 性能对比参考

| 配置 | 索引吞吐 | 查询延迟 (p99) |
|------|:--------:|:--------------:|
| 不做 NUMA 配置 | 基准 | 基准 |
| `--interleave=all` | +5~10% | -5~10% |
| `--cpunodebind=0 --membind=0` | +15~25% | -20~30% |

> 实际收益取决于硬件拓扑、数据规模和查询模式。

## 延伸阅读

- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})
- [生产环境部署]({{< relref "/docs/deployment/install-guide/production-env.md" >}})
- [硬件配置]({{< relref "/docs/deployment/config/hardware.md" >}})

