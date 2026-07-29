---
title: "参数配置"
date: 0001-01-01
description: "系统调优、JVM、Easysearch 配置等。"
summary: "参数配置 #  各层次的配置与调优指南，确保 Easysearch 稳定高效运行。
配置文档 #     文档 说明      节点配置 easysearch.yml、jvm.options、log4j2.properties 配置项详解    系统调优 Linux 内核参数、文件描述符、虚拟内存、网络调优    硬件配置 CPU / 内存 / 磁盘 / 网络选型、容量估算    集群配置 集群设置 API、动态配置项参考    配置优先级 #  Easysearch 按以下优先级读取配置（从高到低）：
 Transient 设置（临时，重启后丢失） Persistent 设置（持久，跨重启保留） 配置文件 easysearch.yml 默认值   推荐做法：节点相关配置放 easysearch.yml，集群级别调优通过 集群设置 API 管理。
 "
---


# 参数配置

各层次的配置与调优指南，确保 Easysearch 稳定高效运行。

## 配置文档

| 文档 | 说明 |
|------|------|
| [节点配置]({{< relref "./node-settings/" >}}) | `easysearch.yml`、`jvm.options`、`log4j2.properties` 配置项详解 |
| [系统调优]({{< relref "./settings" >}}) | Linux 内核参数、文件描述符、虚拟内存、网络调优 |
| [硬件配置]({{< relref "./hardware" >}}) | CPU / 内存 / 磁盘 / 网络选型、容量估算 |
| [集群配置]({{< relref "./configuration" >}}) | 集群设置 API、动态配置项参考 |

## 配置优先级

Easysearch 按以下优先级读取配置（从高到低）：

1. **Transient** 设置（临时，重启后丢失）
2. **Persistent** 设置（持久，跨重启保留）
3. 配置文件 `easysearch.yml`
4. 默认值

> 推荐做法：节点相关配置放 `easysearch.yml`，集群级别调优通过 [集群设置 API]({{< relref "./configuration" >}}) 管理。
