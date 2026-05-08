---
title: "JVM 配置"
date: 0001-01-01
summary: "JVM 配置 #  本页介绍 config/jvm.options 文件中的 JVM 启动参数。这些参数在 Easysearch 启动时由 JVM 读取，修改后需要重启节点生效。
 堆内存设置 #  堆内存是 JVM 配置中最重要的参数。
# 堆大小设为物理内存的 50%，但不超过 31 GB # -Xms 和 -Xmx 必须相同，避免运行时堆调整 -Xms16g -Xmx16g    参数 说明     -Xms JVM 初始堆大小   -Xmx JVM 最大堆大小    设置原则 #   -Xms 和 -Xmx 设为相同值：避免运行时堆大小调整带来的性能波动。 不超过物理内存的 50%：剩余内存留给操作系统文件缓存（Lucene 严重依赖 OS 页面缓存）。 不超过 31 GB（建议设为 31g 或 30g）：超过约 32 GB 后 JVM 无法使用压缩对象指针（Compressed OOPs），实际可用内存反而减少。 不低于 1 GB：过小的堆会导致频繁 GC。  验证压缩指针 #  GET _nodes/stats/jvm?"
---


# JVM 配置

本页介绍 `config/jvm.options` 文件中的 JVM 启动参数。这些参数在 Easysearch 启动时由 JVM 读取，修改后需要**重启节点**生效。

---

## 堆内存设置

堆内存是 JVM 配置中最重要的参数。

```bash
# 堆大小设为物理内存的 50%，但不超过 31 GB
# -Xms 和 -Xmx 必须相同，避免运行时堆调整
-Xms16g
-Xmx16g
```

| 参数 | 说明 |
|------|------|
| `-Xms` | JVM 初始堆大小 |
| `-Xmx` | JVM 最大堆大小 |

### 设置原则

1. **`-Xms` 和 `-Xmx` 设为相同值**：避免运行时堆大小调整带来的性能波动。
2. **不超过物理内存的 50%**：剩余内存留给操作系统文件缓存（Lucene 严重依赖 OS 页面缓存）。
3. **不超过 31 GB**（建议设为 `31g` 或 `30g`）：超过约 32 GB 后 JVM 无法使用压缩对象指针（Compressed OOPs），实际可用内存反而减少。
4. **不低于 1 GB**：过小的堆会导致频繁 GC。

### 验证压缩指针

```bash
GET _nodes/stats/jvm?filter_path=**.using_compressed_ordinary_object_pointers
```

返回 `true` 表示压缩指针生效。

### 通过环境变量覆盖

如果不想修改 `jvm.options` 文件，可以通过环境变量覆盖：

```bash
ES_JAVA_OPTS="-Xms4g -Xmx4g" ./bin/easysearch
```

> 环境变量的优先级高于 `jvm.options` 文件中的设置。

---

## 垃圾回收器

Easysearch 使用的垃圾回收器因 JDK 版本而异：

- **JDK 8-10**：使用 CMS（Concurrent Mark-Sweep）
- **JDK 11+**：默认使用 G1GC

```bash
# JDK 8-10: CMS 垃圾回收器
8-10:-XX:+UseConcMarkSweepGC
8-10:-XX:CMSInitiatingOccupancyFraction=75
8-10:-XX:+UseCMSInitiatingOccupancyOnly

# JDK 11+: G1GC（推荐）
11-:-XX:+UseG1GC
11-:-XX:G1ReservePercent=25
11-:-XX:InitiatingHeapOccupancyPercent=30
```

| 参数 | 说明 |
|------|------|
| `8-10:-XX:+UseConcMarkSweepGC` | JDK 8-10 使用 CMS 垃圾回收器 |
| `8-10:-XX:CMSInitiatingOccupancyFraction=75` | JDK 8-10：老年代占用 75% 时触发 CMS 回收 |
| `11-:-XX:+UseG1GC` | JDK 11+ 使用 G1GC，支持更大的堆和更稳定的暂停时间 |
| `11-:-XX:G1ReservePercent=25` | G1GC 预留 25% 堆用于浮动垃圾 |
| `11-:-XX:InitiatingHeapOccupancyPercent=30` | 堆占用 30% 时启动并发回收 |

> **⚠️ 注意**：版本号前缀（如 `8-10:`、`11-`）表示该参数仅在指定 JDK 版本范围内生效。除非有充分的理由和测试数据，**不建议更改默认 GC 配置**。

---

## OOM 保护

```bash
# 发生 OutOfMemoryError 时生成堆转储文件
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=data

# 发生 OOM 时的错误日志路径
-XX:ErrorFile=logs/hs_err_pid%p.log
```

## Lucene 性能优化（JDK 9+）

```bash
# JDK 9+：启用 Lucene 性能优化模块
# jdk.unsupported：提供 sun.misc.Unsafe 用于低级内存操作
# jdk.management：提供扩展的管理接口
9-:--add-modules=jdk.unsupported,jdk.management
```

## Vector API 支持（JDK 20+）

```bash
# JDK 20+：启用向量 API（孵化功能）用于 SIMD 支持
20-:--add-modules=jdk.incubator.vector
```

> Vector API 可以加速搜索和 Lucene 索引操作，特别是在向量搜索场景中。

---

## OOM 保护（旧语法）

```bash
# 发生 OutOfMemoryError 时生成堆转储文件
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=data

# 发生 OOM 时立即退出 JVM（让进程管理器自动重启）
-XX:+ExitOnOutOfMemoryError
```

| 参数 | 说明 |
|------|------|
| `-XX:+HeapDumpOnOutOfMemoryError` | OOM 时自动生成 heap dump 文件，用于事后分析 |
| `-XX:HeapDumpPath` | Heap dump 文件的存储路径（相对于 `$ES_HOME`） |
| `-XX:+ExitOnOutOfMemoryError` | OOM 时立即退出 JVM，而不是挂起 |

---

## GC 日志

Easysearch 默认启用 GC 日志，不同 JDK 版本的配置方式不同：

### JDK 8 GC 日志

```bash
8:-XX:+PrintGCDetails
8:-XX:+PrintGCDateStamps
8:-XX:+PrintTenuringDistribution
8:-XX:+PrintGCApplicationStoppedTime
8:-Xloggc:logs/gc.log
8:-XX:+UseGCLogFileRotation
8:-XX:NumberOfGCLogFiles=32
8:-XX:GCLogFileSize=64m
```

### JDK 9+ GC 日志

```bash
9-:-Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m
```

| 参数部分 | 说明 |
|------|------|
| `gc*,gc+age=trace,safepoint` | 记录 GC 事件、年龄分布、安全点 |
| `file=logs/gc.log` | 输出到 `logs/gc.log` |
| `utctime,pid,tags` | 日志中包含 UTC 时间、进程 ID、标签 |
| `filecount=32,filesize=64m` | 最多 32 个轮转文件，每个 64 MB |

> GC 日志对排查性能问题非常重要，建议保持默认开启。

---

## 其他常用 JVM 参数

### 临时目录

```bash
# 临时文件目录
-Djava.io.tmpdir=${ES_TMPDIR}
```

### Java Security Manager（JDK 17+）

```bash
# JDK 17+：允许 Java Security Manager
17-:-Djava.security.manager=allow
```

### Locale 配置

```bash
# JDK 9-20：使用 SPI 和系统兼容性 Provider
9-20:-Djava.locale.providers=SPI,COMPAT

# JDK 21+：使用 SPI 和 CLDR Provider
21-:-Djava.locale.providers=SPI,CLDR
```

### 编译优化（JDK 23+）

```bash
# JDK 23：修复 MethodHandle 性能问题
23:-XX:CompileCommand=dontinline,java/lang/invoke/MethodHandle.setAsTypeCache
23:-XX:CompileCommand=dontinline,java/lang/invoke/MethodHandle.asTypeUncached
```

### Native Access（JDK 19+）

```bash
# JDK 19+：为所有未命名模块启用原生访问
19-:--enable-native-access=ALL-UNNAMED

# JDK 22+：禁用向量 API 警告
22-:-Djdk.incubator.vector.disableWarnings=true
```

| 参数 | 说明 |
|------|------|
| `-Djava.io.tmpdir` | JVM 临时文件目录 |
| `-Djava.security.manager` | JDK 17+ 的安全管理器（allow 表示允许使用） |
| `-Djava.locale.providers` | JVM 使用的 Locale 提供者 |

---

## jvm.options 文件格式

`jvm.options` 文件中每行一个 JVM 参数，支持版本条件：

```bash
# 这是注释

# 无条件参数（所有 JVM 版本生效）
-Xms16g
-Xmx16g

# 版本条件参数
8-10:-XX:+UseConcMarkSweepGC       # JDK 8~10
11-:-XX:+UseG1GC                   # JDK 11+
9-20:-Djava.locale.providers=SPI,COMPAT  # JDK 9~20
21-:-Djava.locale.providers=SPI,CLDR     # JDK 21+
```

### 版本条件语法

| 格式 | 说明 | 示例 |
|------|------|------|
| 无前缀 | 所有版本生效 | `-Xms16g` |
| `N-` | JDK N 及以上版本 | `11-:-XX:+UseG1GC` (JDK 11+) |
| `N-M` | JDK N 到 M 版本 | `8-10:-XX:+UseConcMarkSweepGC` (JDK 8-10) |

- 以 `#` 开头的行是注释。
- 空行会被忽略。

---

## jvm.options.d 目录

除了主 `jvm.options` 文件外，Easysearch 还会读取 `config/jvm.options.d/` 目录下所有以 `.options` 结尾的文件。每个文件的格式与主 `jvm.options` 完全相同。

### 使用场景

- **自定义参数与默认参数分离**：将自定义 JVM 参数放在独立文件中，避免修改默认的 `jvm.options` 文件，便于升级时保留定制配置
- **容器化部署**：通过挂载配置文件到 `jvm.options.d/` 目录，在不修改镜像的情况下调整 JVM 参数
- **按关注点组织**：将不同类型的参数（如 GC、调试、安全）放在不同文件中

### 示例

创建自定义堆内存配置：

```bash
# config/jvm.options.d/heap.options
-Xms16g
-Xmx16g
```

创建调试参数文件：

```bash
# config/jvm.options.d/debug.options
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/log/easysearch
-XX:ErrorFile=/var/log/easysearch/hs_err_pid%p.log
```

### 加载顺序

1. 先读取 `config/jvm.options` 主文件
2. 再按文件名字母顺序读取 `config/jvm.options.d/*.options`
3. 如果同一参数出现多次，后加载的值会覆盖先加载的值

> **注意**：`jvm.options.d/` 目录中的文件必须以 `.options` 为扩展名，其他文件会被忽略。

---

## 配置示例

### 生产环境（64 GB 服务器，JDK 11+）

```bash
# 堆内存：物理内存的 50%，不超过 31 GB
-Xms31g
-Xmx31g

# G1GC（JDK 11+）
11-:-XX:+UseG1GC
11-:-XX:G1ReservePercent=25
11-:-XX:InitiatingHeapOccupancyPercent=30

# OOM 保护
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=data
-XX:ErrorFile=logs/hs_err_pid%p.log

# GC 日志
9-:-Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m

# 其他
-Djava.io.tmpdir=${ES_TMPDIR}
```

### 开发环境（8 GB 笔记本）

```bash
-Xms1g
-Xmx1g
```

### 传统 CMS 环境（JDK 8-10，64 GB 服务器）

```bash
-Xms31g
-Xmx31g

# CMS GC（JDK 8-10）
8-10:-XX:+UseConcMarkSweepGC
8-10:-XX:CMSInitiatingOccupancyFraction=75
8-10:-XX:+UseCMSInitiatingOccupancyOnly

# OOM 保护
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=data
-XX:ErrorFile=logs/hs_err_pid%p.log
```

---

## 常见问题

### 启动失败：堆内存分配失败

```
Initial heap size set to an invalid value
```

通常是因为 `-Xms` 或 `-Xmx` 超过了物理内存或操作系统限制。

### 频繁 Full GC

- 检查堆大小是否足够。
- 使用 `GET _nodes/stats/jvm` 查看堆使用率。
- 分析 GC 日志（`logs/gc.log`）定位原因。

### 压缩指针未启用

堆大小超过约 32 GB 时压缩指针失效。建议设为 `31g` 或以下。

---

## 延伸阅读

- [内存与缓存]({{< relref "./memory.md" >}}) — `bootstrap.memory_lock` 与缓存配置
- [硬件配置]({{< relref "../hardware.md" >}}) — 内存规格选型建议
- [系统调优]({{< relref "../settings.md" >}}) — `memlock`、文件描述符等 OS 参数

