---
title: "日志配置"
date: 0001-01-01
summary: "日志配置 #  本页介绍 config/log4j2.properties 日志配置文件以及慢日志设置。
 日志文件说明 #  Easysearch 使用 Log4j 2 作为日志框架。日志文件默认存储在 ${ES_HOME}/logs/ 目录下（可通过 path.logs 自定义）。
   文件 说明     ${cluster.name}.log 主日志文件，记录集群运行的核心事件   ${cluster.name}_server.json JSON 格式日志，便于日志采集工具（如 Filebeat）解析   ${cluster.name}_deprecation.log API 弃用警告日志，记录使用了即将移除的 API   ${cluster.name}_slowlog.log 慢查询和慢索引日志   gc.log JVM 垃圾回收日志（由 JVM 参数控制）     log4j2.properties #  config/log4j2.properties 控制日志行为。Easysearch 的默认配置已经比较完善，通常无需修改。
默认日志级别 #  默认日志级别为 INFO，在 log4j2.properties 中可以修改："
---


# 日志配置

本页介绍 `config/log4j2.properties` 日志配置文件以及慢日志设置。

---

## 日志文件说明

Easysearch 使用 Log4j 2 作为日志框架。日志文件默认存储在 `${ES_HOME}/logs/` 目录下（可通过 `path.logs` 自定义）。

| 文件 | 说明 |
|------|------|
| `${cluster.name}.log` | 主日志文件，记录集群运行的核心事件 |
| `${cluster.name}_server.json` | JSON 格式日志，便于日志采集工具（如 Filebeat）解析 |
| `${cluster.name}_deprecation.log` | API 弃用警告日志，记录使用了即将移除的 API |
| `${cluster.name}_slowlog.log` | 慢查询和慢索引日志 |
| `gc.log` | JVM 垃圾回收日志（由 JVM 参数控制） |

---

## log4j2.properties

`config/log4j2.properties` 控制日志行为。Easysearch 的默认配置已经比较完善，通常无需修改。

### 默认日志级别

默认日志级别为 `INFO`，在 `log4j2.properties` 中可以修改：

```properties
# 根日志级别
rootLogger.level = info

# 为特定包设置不同级别
logger.discovery.name = org.easysearch.discovery
logger.discovery.level = warn
```

### 日志级别

| 级别 | 说明 |
|------|------|
| `TRACE` | 最详细，会产生大量日志 |
| `DEBUG` | 调试信息 |
| `INFO` | 正常运行信息（默认） |
| `WARN` | 警告信息 |
| `ERROR` | 错误信息 |
| `FATAL` | 致命错误 |

### 日志轮转

默认配置包含日志轮转，防止日志文件无限增长：

```properties
# 按大小和日期轮转
appender.rolling.type = RollingFile
appender.rolling.policies.size.size = 128MB
```

---

## 动态调整日志级别

**推荐做法**：通过集群设置 API 动态修改日志级别，无需修改文件、无需重启。

### 开启 DEBUG 日志

```json
PUT /_cluster/settings
{
  "transient": {
    "logger.org.easysearch.discovery": "DEBUG"
  }
}
```

### 恢复默认级别

```json
PUT /_cluster/settings
{
  "transient": {
    "logger.org.easysearch.discovery": null
  }
}
```

### 常用 Logger 名称

| Logger | 用途 |
|--------|------|
| `org.easysearch.discovery` | 集群发现与主节点选举 |
| `org.easysearch.cluster.service` | 集群状态更新 |
| `org.easysearch.index` | 索引操作 |
| `org.easysearch.gateway` | 分片恢复 |
| `org.easysearch.transport` | 节点间通信 |
| `org.easysearch.action` | API 请求处理 |
| `org.easysearch.indices.recovery` | 分片恢复详情 |

---

## 慢日志

慢日志帮助发现性能问题，记录执行时间超过阈值的搜索和索引操作。

### 慢日志配置

慢日志是**索引级别**的设置，可以通过 API 动态修改：

```json
PUT /my-index/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s",
  "index.search.slowlog.threshold.query.debug": "2s",
  "index.search.slowlog.threshold.query.trace": "500ms",

  "index.search.slowlog.threshold.fetch.warn": "1s",
  "index.search.slowlog.threshold.fetch.info": "800ms",
  "index.search.slowlog.threshold.fetch.debug": "500ms",
  "index.search.slowlog.threshold.fetch.trace": "200ms",

  "index.indexing.slowlog.threshold.index.warn": "10s",
  "index.indexing.slowlog.threshold.index.info": "5s",
  "index.indexing.slowlog.threshold.index.debug": "2s",
  "index.indexing.slowlog.threshold.index.trace": "500ms"
}
```

### 慢日志参数说明

| 参数前缀 | 说明 |
|----------|------|
| `index.search.slowlog.threshold.query.*` | 搜索查询阶段（query phase）的慢日志阈值 |
| `index.search.slowlog.threshold.fetch.*` | 搜索获取阶段（fetch phase）的慢日志阈值 |
| `index.indexing.slowlog.threshold.index.*` | 索引写入操作的慢日志阈值 |

每种类型支持 4 个级别：`warn`、`info`、`debug`、`trace`。只有超过对应阈值的操作才会以该级别记录到慢日志。

### 为所有索引设置慢日志

使用索引模板为所有新建索引自动配置慢日志：

```json
PUT /_template/slowlog_template
{
  "index_patterns": ["*"],
  "settings": {
    "index.search.slowlog.threshold.query.warn": "10s",
    "index.search.slowlog.threshold.query.info": "5s",
    "index.indexing.slowlog.threshold.index.warn": "10s"
  }
}
```

### 慢日志输出位置

慢日志输出到 `${path.logs}/${cluster.name}_slowlog.log`。

---

## 弃用日志

弃用日志记录你使用的即将被移除的 API 或功能。升级前应重点检查这些日志。

- 默认输出到 `${cluster.name}_deprecation.log`。
- 也可通过 API 查看：

```json
GET /_cluster/settings?flat_settings=true&filter_path=**.deprecation
```

---

## 日志排查最佳实践

1. **平时保持 INFO 级别**，避免日志量过大。
2. **排查问题时**通过 API 临时开启 DEBUG，排查完后立即关闭。
3. **保持 GC 日志开启**（`jvm.options` 中默认已配置），便于事后分析性能问题。
4. **配置日志采集**：使用 Filebeat 将日志发送到独立的监控集群。
5. **定期清理**：确保日志目录有足够空间，检查日志轮转是否正常。

---

## 延伸阅读

- [Log4j2 日志配置]({{< relref "./log4j2.md" >}}) — log4j2.properties 文件配置细节
- [JVM 配置]({{< relref "./jvm.md" >}}) — GC 日志参数
- [路径配置]({{< relref "./path.md" >}}) — `path.logs` 日志目录
- [集群配置]({{< relref "../configuration.md" >}}) — 动态日志级别调整

