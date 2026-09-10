---
title: "高阶配置与调优"
date: 0001-01-01
description: "NUMA、NVME、RAID、国密、TLS 等高阶配置"
summary: "高阶配置与调优 #  面向生产环境的高级配置与性能调优指南。
系统级配置 #     文档 内容     NUMA 配置 NUMA 优化配置   NVME 配置 NVME 存储优化   RAID 配置 RAID 阵列配置   磁盘加密 数据安全加密   国密配置 国密算法支持   TLS 安全配置 SSL/TLS 安全通信    高级调优 #     文档 内容     集群协调调优 选举、故障检测、集群状态发布参数调优   Ingest Pipeline 配置 Pipeline 管理、Processor 使用、性能调优   索引排序 索引排序配置与提前终止优化   高级配置参数 集群、节点、索引和传输层配置参数分类参考     ❓ 常见问题解答 #  Q1: Easysearch 服务启动很慢，如何排查？ #  常见原因："
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
| [高级配置参数]({{< relref "./advanced-settings.md" >}}) | 集群、节点、索引和传输层配置参数分类参考 |

---

## ❓ 常见问题解答

### Q1: Easysearch 服务启动很慢，如何排查？

**常见原因：**

1. JVM 堆设置过大，`AlwaysPreTouch` 在启动期间初始化全部堆内存页
2. 节点发现或集群引导配置错误
3. 分片恢复、索引加载或磁盘 I/O 较慢

**诊断步骤：**

```bash
# 检查实际生效的 JVM 参数
jcmd <pid> VM.flags

# 将路径替换为当前节点的 ${path.logs}/<cluster.name>.log
ES_LOG_FILE="/actual/path/to/path.logs/<cluster.name>.log"
grep -E "master not discovered|recovered|recovery|started" "$ES_LOG_FILE"
```

节点启动后，可继续检查仍在进行的分片恢复：

```text
GET /_cat/recovery?v&active_only=true
```

Easysearch 启动器已经启用 `AlwaysPreTouch`。它可以改善运行期间的内存访问稳定性，但会增加大堆节点的启动耗时，不应将其
作为启动加速参数重复添加。调整堆内存时应保持 `-Xms` 与 `-Xmx` 相同，并根据物理内存和工作负载确定大小，不能直接套用固定值。

`discovery.seed_hosts` 用于节点发现，并不是性能调优参数。应确认其中的地址可以解析和连通，并检查
`cluster.initial_master_nodes` 是否只用于新集群的首次引导。详见[集群协调调优]({{< relref "./cluster-coordination.md" >}})和
[JVM 配置]({{< relref "../config/node-settings/jvm.md" >}})。

---

### Q2: 查询响应时间很长（>1s），如何优化？

**快速诊断：**

```text
# 启用慢查询日志
PUT /my-index/_settings
{
  "index.search.slowlog.threshold.query.warn": "500ms"
}
```

慢日志默认写入 `${path.logs}/<cluster.name>_index_search_slowlog.json` 和
`${path.logs}/<cluster.name>_index_search_slowlog.log`，不会自动写入 Easysearch 索引。可以在节点上直接检查对应文件：

```bash
# 将路径替换为当前节点实际配置的 path.logs
ES_LOG_DIR="/actual/path/to/path.logs"
ls "$ES_LOG_DIR"/*_index_search_slowlog.json "$ES_LOG_DIR"/*_index_search_slowlog.log
```

若需让所有新索引启用慢日志，应通过索引模板配置。完整配置方法见
[日志配置]({{< relref "../config/node-settings/logging.md" >}})。

**常见原因和解决方案：**

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 缓存未命中 | 相同查询每次都慢 | 检查查询/请求缓存统计和查询是否满足缓存条件；查询缓存默认已启用 |
| Bool 查询过复杂 | CPU 占用高 | 展平查询、使用 terms |
| 高成本排序 | Fetch 或排序阶段耗时高 | 减少排序数据量；固定排序模式可在新索引上验证索引排序并重建数据 |
| GC 停顿 | 间歇性延迟 | 检查 GC 日志、堆占用和对象分配，再评估 JVM 调整 |
| 线程池拒绝 | 查询超时 | 定位被拒绝的线程池和请求，限制并发、降低单请求成本或扩容节点 |

---

### Q3: 磁盘占用快速增长，如何控制？

**诊断方法：**

```text
# 查看分片大小
GET /_cat/shards?v&h=index,shard,prirep,state,store,node&s=store:desc

# 查看索引大小
GET /_cat/indices?v&h=index,store.size&s=store.size:desc
```

**常见原因和解决方案：**

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 活跃索引段数较多 | 小段多、合并持续运行 | 优先等待后台自动合并，不要对仍在写入的索引执行 force merge |
| Translog 占用较大 | `_stats/translog` 显示大小持续增长 | 检查 flush 阈值、保留租约和恢复活动；必要时在维护窗口执行 flush，而不是 refresh |
| 编码器压缩率较低 | `_source`、stored fields 占用较高 | 为新建或重建的冷数据索引评估 `best_compression`，并验证写入和查询开销 |
| 只读索引包含大量删除文档 | 删除文档比例和磁盘占用较高 | 确保磁盘余量后，对只读索引评估 `only_expunge_deletes` |

Force merge 只适合不再写入的只读或归档索引，并可能在执行期间临时增加磁盘占用。详见
[Refresh、Flush 与 Force Merge]({{< relref "../../operations/data-management/refresh-flush-forcemerge.md" >}})。

---

### Q4: 内存占用持续增长，最后 OOM，怎么办？

**首先保留现场并确认内存消耗来源：**

```text
GET /_nodes/stats/jvm,breaker?filter_path=nodes.*.jvm.mem,nodes.*.breakers

GET /_stats/fielddata,query_cache,request_cache,segments
```

发行包默认已经启用 `-XX:+HeapDumpOnOutOfMemoryError` 并配置 heap dump 路径。应确认目标目录存在、空间足够且受到访问控制，
再分析 OOM 时生成的 dump。若必须在线生成 dump，应在维护窗口执行，并预留至少与堆大小相当的磁盘空间：

```bash
jcmd <pid> GC.heap_dump /path/to/heap.hprof
```

生成 heap dump 可能造成长时间停顿，文件也可能包含敏感数据。不要在未确认物理内存、文件缓存、容器限制和实际对象占用前直接
增加堆。确需调整时，必须同时修改 `-Xms` 和 `-Xmx`，并遵守[JVM 配置]({{< relref "../config/node-settings/jvm.md" >}})
中的容量约束。

---

### Q5: 集群不稳定，频繁出现节点掉线，怎么排查？

**诊断步骤：**

```text
# 查看集群健康状态
GET /_cluster/health?pretty

# 查看节点状态
GET /_cat/nodes?v&h=name,master,node.role,cpu,heap.percent,load_1m,disk.used_percent
```

```bash
# 将路径替换为当前节点的 ${path.logs}/<cluster.name>.log
ES_LOG_FILE="/actual/path/to/path.logs/<cluster.name>.log"
grep -Ei "master not discovered|node-left|disconnected|gc|circuit_breaking" "$ES_LOG_FILE" | tail -100
```

**常见原因和解决方案：**

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 主节点发现超时 | 频繁重选主 | 先检查 DNS、网络、GC 和节点负载；确认误判后再评估故障检测参数 |
| GC 停顿长 | 节点假死 | 优化 JVM 配置 |
| 网络不稳定 | 间歇性连接问题 | 修复丢包、延迟、DNS、防火墙或连接跟踪问题 |
| 线程池队列满 | 请求拒绝 | 定位请求来源，实施限流、背压、查询优化或节点扩容 |
| 磁盘空间不足 | 触发磁盘水位、分片无法分配或索引变为只读 | 限制新增写入并扩容；核对快照和保留策略后，只清理已确认过期的数据 |

`cluster.fault_detection.*` 属于静态节点配置。增大 timeout 或 retry count 会延迟真实故障的发现，只能在确认网络抖动或长时间
GC 是误判原因后修改，并需要重启节点。不要仅为消除掉线现象扩大故障检测阈值或线程池。

删除索引前必须确认索引用途、别名或数据流关系、保留要求，以及可用快照是否已成功完成并能够恢复。不要在未展开并核对实际索引
清单时使用通配符删除。

---

### Q6: CPU 占用率很高（>80%），如何降低？

短时 CPU 峰值并不一定表示故障。应结合查询延迟、写入吞吐、线程池拒绝和持续时间，确认是否存在资源饱和。

**诊断方法：**

```text
# 查看热线程
GET /_nodes/hot_threads?threads=5&type=cpu&ignore_idle_threads=true

# 查看线程池统计
GET /_cat/thread_pool?v&h=node_name,name,active,queue,rejected

# 对照 JVM GC、查询、写入和合并统计
GET /_nodes/stats/process,jvm,indices?filter_path=nodes.*.process.cpu,nodes.*.jvm.gc,nodes.*.indices.search,nodes.*.indices.indexing,nodes.*.indices.merges
```

**常见原因和解决方案：**

| 原因 | 症状 | 解决方案 |
|------|------|---------|
| 复杂查询或聚合 | search 线程持续占用 CPU | 使用 Profile API 定位高成本阶段，减少脚本、前导通配符、高基数聚合或过大的结果集 |
| 集中写入触发大量合并 | merge 线程和磁盘 I/O 同时升高 | 削平写入峰值或提升存储能力，默认保留 merge 自动节流，不要直接修改 merge policy |
| GC 频繁 | GC 线程 CPU 高且暂停增加 | 检查 GC 日志、分配速率、缓存和 breaker，再根据证据调整工作负载或 JVM |
| 高成本排序 | search 线程 CPU 高 | 减少命中文档和排序字段；只有固定排序长期匹配时，才在新建索引上验证索引排序并重建数据 |

---

### Q7: 如何选择合适的 GC 算法？

Easysearch 2.4.0 要求 JDK 21 或更高版本，发行配置默认使用 G1GC。堆大小本身不能作为切换 GC 的唯一依据；通常应保留发行包
默认设置，并先通过 GC 日志确认吞吐、暂停时间和对象分配问题。

```bash
jcmd <pid> VM.flags
jcmd <pid> GC.heap_info
```

CMS 不适用于当前支持的运行时。ZGC 等其他收集器也不是默认配置，只能在固定 JDK、真实业务负载和完整稳定性测试下验证后使用，
不能仅因为堆大于某个阈值直接切换。详细设置见[JVM 配置]({{< relref "../config/node-settings/jvm.md" >}})。

---



## 相关文档

- [基础配置]({{< relref "../config/configuration.md" >}}) - easysearch.yml 配置说明
- [节点配置]({{< relref "../config/node-settings/" >}}) - 节点级别的配置指南
- [集群协调调优]({{< relref "./cluster-coordination.md" >}}) - 选举与故障检测参数
- [Ingest Pipeline 配置]({{< relref "./ingest-pipeline-config.md" >}}) - Pipeline 高级配置
- [索引排序]({{< relref "./index-sorting.md" >}}) - 索引排序与查询加速
- [高级配置参数]({{< relref "./advanced-settings.md" >}}) - 集群、节点、索引和传输层配置参数分类参考
