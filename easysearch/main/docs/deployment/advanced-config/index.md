---
title: "高阶配置与调优"
date: 0001-01-01
description: "NUMA、NVME、RAID、国密、TLS 等高阶配置"
summary: "高阶配置与调优 #  面向生产环境的高级配置与性能调优指南。
系统级配置 #     文档 内容      NUMA 配置 NUMA 优化配置    NVME 配置 NVME 存储优化    RAID 配置 RAID 阵列配置    磁盘加密 数据安全加密    国密配置 国密算法支持    TLS 安全配置 SSL/TLS 安全通信    高级调优 #     文档 内容      集群协调调优 选举、故障检测、集群状态发布参数调优    Ingest Pipeline 配置 Pipeline 管理、Processor 使用、性能调优    索引排序 索引排序配置与提前终止优化    高级配置参数 484 个配置参数完整参考     ❓ 常见问题解答 #  Q1: Easysearch 服务启动很慢，如何加速？ #  原因分析："
---


# 高阶配置与调优

面向生产环境的高级配置与性能调优指南。

## 系统级配置

| 文档 | 内容 |
|------|------|
| [NUMA 配置]({{< relref "./numa.md" >}}) | NUMA 优化配置 |
| [NVME 配置]({{< relref "./nvme.md" >}}) | NVME 存储优化 |
| [RAID 配置]({{< relref "./raid.md" >}}) | RAID 阵列配置 |
| [磁盘加密]({{< relref "./disk-encryption.md" >}}) | 数据安全加密 |
| [国密配置]({{< relref "./guomi.md" >}}) | 国密算法支持 |
| [TLS 安全配置]({{< relref "./tls-secure.md" >}}) | SSL/TLS 安全通信 |

## 高级调优

| 文档 | 内容 |
|------|------|
| [集群协调调优]({{< relref "./cluster-coordination.md" >}}) | 选举、故障检测、集群状态发布参数调优 |
| [Ingest Pipeline 配置]({{< relref "./ingest-pipeline-config.md" >}}) | Pipeline 管理、Processor 使用、性能调优 |
| [索引排序]({{< relref "./index-sorting.md" >}}) | 索引排序配置与提前终止优化 |
| [高级配置参数]({{< relref "./advanced-settings.md" >}}) | 484 个配置参数完整参考 |

---

## ❓ 常见问题解答

### Q1: Easysearch 服务启动很慢，如何加速？

**原因分析：**
1. JVM 参数不合理（堆大小、GC 算法）
2. 集群发现耗时
3. 索引加载耗时

**解决步骤：**

```yaml
# 1. 检查 JVM 配置
jcmd <pid> VM.flags | grep -E "Xmx|Xms|UseG1GC"

# 2. 优化 JVM 启动参数
-Xms16g
-Xmx16g
-XX:+UseG1GC
-XX:+AlwaysPreTouch  # 启动时预分配内存

# 3. 检查集群发现配置
discovery.seed_hosts: [node1, node2]
```

---

### Q2: 查询响应时间很长（>1s），如何优化？

**快速诊断：**

```bash
# 启用慢查询日志
PUT /_cluster/settings
{
  "persistent": {
    "index.search.slowlog.threshold.query.warn": "500ms"
  }
}

# 获取慢查询
GET /.slowlog-*/_search?size=10&sort=timestamp:desc
```

**常见原因和解决方案：**

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 缓存未命中 | 相同查询每次都慢 | 启用查询缓存 |
| Bool 查询过复杂 | CPU 占用高 | 展平查询、使用 terms |
| 排序字段多值 | 排序时间长 | 使用索引排序 |
| GC 停顿 | 间歇性延迟 | 调整 GC 算法 |
| 线程池拒绝 | 查询超时 | 增加线程池大小 |

---

### Q3: 磁盘占用快速增长，如何控制？

**诊断方法：**

```bash
# 查看分片大小
GET /_cat/shards?v&h=index,shard,size,store

# 查看索引大小
GET /_cat/indices?v&h=index,store.size&sort=store.size:desc
```

**常见原因和解决方案：**

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 索引未合并 | 段数多、磁盘占用大 | 强制合并 |
| Translog 未清理 | translog.log 很大 | 刷新索引 |
| 编码器选择不当 | 压缩率低 | 使用 best_compression |
| 索引排序过多 | 索引大小 +50% | 减少排序字段数 |

---

### Q4: 内存占用持续增长，最后 OOM，怎么办？

**紧急处理：**

```yaml
# 立即增加堆大小（重启生效）
-Xmx32g  # 从当前值增加 50%

# 启用 OOM 诊断
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./dumps/
```

**根本原因诊断：**

```bash
# 监控缓存大小
GET /_stats/query_cache

# 分析堆转储
jmap -dump:live,format=b,file=heap.bin <pid>
```

---

### Q5: 集群不稳定，频繁出现节点掉线，怎么排查？

**诊断步骤：**

```bash
# 查看集群健康状态
GET /_cluster/health?pretty

# 查看节点状态
GET /_cat/nodes?v&h=name,status,load,heap.percent,cpu

# 查看主节点日志
grep -i "master" logs/easysearch.log | tail -20
```

**常见原因和解决方案：**

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 主节点发现超时 | 频繁重选主 | 调整心跳超时参数 |
| GC 停顿长 | 节点假死 | 优化 JVM 配置 |
| 网络不稳定 | 间歇性连接问题 | 增加心跳重试次数 |
| 线程池队列满 | 请求拒绝 | 增加线程池大小 |
| 磁盘空间不足 | 索引无法写入 | 扩展磁盘或删除旧索引 |

---

### Q6: CPU 占用率很高（>80%），如何降低？

**诊断方法：**

```bash
# 查看热线程
GET /_nodes/hot_threads?threads=5

# 查看线程池统计
GET /_cat/thread_pool?v&h=name,active,queue,rejected
```

**常见原因和解决方案：**

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 复杂查询 | search 线程 CPU 高 | 简化查询结构 |
| 合并操作 | merge 线程 CPU 高 | 控制合并频率 |
| GC 频繁 | GC 线程 CPU 高 | 优化 JVM 堆大小 |
| 排序操作 | search/merge CPU 高 | 使用索引排序 |

---

### Q7: 如何选择合适的 GC 算法？

**快速决策表：**

| Java 版本 | 堆大小 | 推荐算法 | 理由 |
|----------|--------|---------|------|
| Java 8-11 | <8GB | CMS 或 G1GC | 低延迟 |
| Java 8-11 | >8GB | G1GC | 自适应 |
| Java 12-14 | 任意 | G1GC | 成熟稳定 |
| Java 15+ | <16GB | G1GC | 通用 |
| Java 15+ | >16GB | ZGC | 极低停顿 |

---



## 相关文档

- [基础配置]({{< relref "../config/configuration.md" >}}) - elasticsearch.yml 配置说明
- [节点配置]({{< relref "../config/node-settings/" >}}) - 节点级别的配置指南
- [集群协调调优]({{< relref "./cluster-coordination.md" >}}) - 选举与故障检测参数
- [Ingest Pipeline 配置]({{< relref "./ingest-pipeline-config.md" >}}) - Pipeline 高级配置
- [索引排序]({{< relref "./index-sorting.md" >}}) - 索引排序与查询加速
- [高级配置参数]({{< relref "./advanced-settings.md" >}}) - 484 个配置参数完整参考
