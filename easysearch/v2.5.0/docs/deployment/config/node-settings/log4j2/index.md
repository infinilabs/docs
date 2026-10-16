---
title: "Log4j2 日志配置"
date: 0001-01-01
summary: "Log4j2 日志配置 #  config/log4j2.properties 是 Easysearch 的日志配置文件，控制日志输出格式、级别、文件轮转等。
 文件位置与格式 #  # 默认位置 $ES_HOME/config/log4j2.properties 该文件使用 Log4j2 的 properties 格式，配置 appenders（日志目标）、loggers（日志器）和日志级别。
修改后无需重启，Easysearch 会自动重新加载配置。
 核心概念 #  Appenders（日志输出目标） #  定义日志输出到哪里，有以下类型：
   Appender 说明     Console 控制台输出（测试环境使用）   RollingFile 滚动文件输出（轮转、自动删除旧文件）    Loggers（日志器） #  根据组件或模块设置不同的日志级别。根 logger (rootLogger) 作用于全局。
日志级别 #     级别 说明     TRACE 最详细   DEBUG 调试信息   INFO 一般信息（默认）   WARN 警告   ERROR 错误   FATAL 致命错误     默认日志文件 #  Easysearch 默认配置会生成以下日志文件（存储在 ${ES_HOME}/logs/）："
---


# Log4j2 日志配置

`config/log4j2.properties` 是 Easysearch 的日志配置文件，控制日志输出格式、级别、文件轮转等。

---

## 文件位置与格式

```bash
# 默认位置
$ES_HOME/config/log4j2.properties
```

该文件使用 Log4j2 的 properties 格式，配置 appenders（日志目标）、loggers（日志器）和日志级别。

修改后**无需重启**，Easysearch 会自动重新加载配置。

---

## 核心概念

### Appenders（日志输出目标）

定义日志输出到哪里，有以下类型：

| Appender | 说明 |
|----------|------|
| `Console` | 控制台输出（测试环境使用） |
| `RollingFile` | 滚动文件输出（轮转、自动删除旧文件） |

### Loggers（日志器）

根据组件或模块设置不同的日志级别。根 logger (`rootLogger`) 作用于全局。

### 日志级别

| 级别 | 说明 |
|------|------|
| `TRACE` | 最详细 |
| `DEBUG` | 调试信息 |
| `INFO` | 一般信息（默认） |
| `WARN` | 警告 |
| `ERROR` | 错误 |
| `FATAL` | 致命错误 |

---

## 默认日志文件

Easysearch 默认配置会生成以下日志文件（存储在 `${ES_HOME}/logs/`）：

| 日志文件 | 用途 |
|---------|------|
| `<cluster-name>.log` | 主日志（纯文本，旧格式） |
| `<cluster-name>_server.json` | 主日志（JSON 格式，便于采集） |
| `<cluster-name>_deprecation.log` | 已废弃 API 警告 |
| `<cluster-name>_deprecation.json` | 已废弃 API 警告（JSON 格式） |
| `<cluster-name>_index_search_slowlog.log` | 搜索慢查询日志 |
| `<cluster-name>_index_search_slowlog.json` | 搜索慢查询日志（JSON 格式） |
| `<cluster-name>_index_indexing_slowlog.log` | 索引慢日志 |
| `<cluster-name>_index_indexing_slowlog.json` | 索引慢日志（JSON 格式） |

---

## 常见配置修改

### 调整根日志级别

```properties
# 修改这一行，可选：TRACE, DEBUG, INFO, WARN, ERROR, FATAL
rootLogger.level=info
```

**示例：启用 DEBUG 日志**

```properties
rootLogger.level=debug
```

> 此方法会增加日志文件大小，建议在排查问题时临时使用。

### 调整特定模块日志级别

```properties
# 为特定模块设置日志级别
logger.discovery.name=org.easysearch.discovery
logger.discovery.level=debug

logger.cluster.name=org.easysearch.cluster
logger.cluster.level=debug
```

### 禁用特定日志

```properties
# 禁用某个模块的日志输出
logger.quiet_module.name=org.easysearch.some.module
logger.quiet_module.level=off
logger.quiet_module.appenderRef.console.ref=console
logger.quiet_module.additivity=false
```

### 日志轮转设置

默认配置中，日志文件按大小（128 MB）和时间（每天）自动轮转：

```properties
# 按大小轮转
appender.rolling.policies.size.size=128MB

# 按时间轮转（每天）
appender.rolling.policies.time.type=TimeBasedTriggeringPolicy
appender.rolling.policies.time.interval=1
appender.rolling.policies.time.modulate=true
```

**修改轮转大小（例：改为 256 MB）**

```properties
appender.rolling.policies.size.size=256MB
```

### 日志自动删除策略

日志累积超过指定大小时自动删除最旧的文件：

```properties
# 日志文件总大小超过 2GB 时，删除最旧的文件
appender.rolling.strategy.action.condition.nested_condition.exceeds=2GB
```

---

## 动态调整日志级别（推荐）

如果不想重启或编辑文件，可以使用 **集群设置 API** 动态调整日志级别。详见 [日志配置 - 动态调整日志级别]({{< relref "./logging.md#动态调整日志级别" >}})。

**恢复为默认值**

```json
PUT /_cluster/settings
{
  "transient": {
    "logger.org.easysearch.discovery": null
  }
}
```

> 此方法修改仅对当前集群状态生效，重启节点后失效。对于持久修改，需编辑 `log4j2.properties`。

---

## JSON 格式日志

Easysearch 的默认配置使用 **JSON 格式日志**，便于日志采集和分析：

**JSON 日志示例**

```json
{
  "@timestamp": "2026-02-18T10:30:45.123Z",
  "log.level": "INFO",
  "message": "started",
  "node.name": "node-1",
  "cluster.name": "my-cluster",
  "elasticsearch.version": "1.3.0"
}
```

**JSON 日志的好处**：
- 结构化便于机器解析
- 易于集成到 ELK/日志系统
- 支持强大的日志聚合和搜索

---

## 慢日志配置

### 搜索慢日志

搜索慢日志在 `<cluster-name>_index_search_slowlog.log` 中。慢查询阈值通过索引设置配置（详见[日志配置]({{< relref "./logging.md" >}})）：

```json
PUT /my-index/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s"
}
```

### 索引慢日志

索引慢日志在 `<cluster-name>_index_indexing_slowlog.log` 中：

```json
PUT /my-index/_settings
{
  "index.indexing.slowlog.threshold.index.warn": "10s",
  "index.indexing.slowlog.threshold.index.info": "5s"
}
```

---

## 故障排查

### 日志文件爆满

**症状**：磁盘空间不足

**原因**：
- 日志级别设置过高（如 DEBUG）
- 轮转策略配置不当
- 日志没有及时删除

**解决**：
1. 检查日志级别是否过高
2. 确认轮转大小和删除策略
3. 手动删除旧日志：`rm logs/my-cluster-*.log.gz`

### 找不到特定的日志信息

**症状**：想找某个操作的日志但找不到

**原因**：
- 日志级别不足（如操作在 DEBUG 级别才输出）
- 操作来自的模块日志被禁用

**解决**：
1. 暂时提高日志级别（使用集群设置 API）
2. 指定模块进行调试
3. 查看相关模块的 logger 配置

### 日志格式乱码

**原因**：字符编码不匹配

**解决**：
- 确保日志查看工具支持 UTF-8
- 检查系统 locale：`echo $LANG`

---

## 最佳实践

1. **生产环境**：保持 INFO 级别，仅在排查问题时临时调整
2. **使用 JSON 日志**：便于采集和分析，不要禁用 `*_server.json` appender
3. **定期清理**：启用自动轮转和删除，避免日志爆满
4. **集中管理**：使用日志采集工具（Filebeat、Logstash）将日志聚合到中央日志系统
5. **监控关键日志**：对 WARN、ERROR、FATAL 级别设置告警

---

## 延伸阅读

- [日志配置]({{< relref "./logging.md" >}}) — 日志系统概览、慢日志配置、Logger 名称
- [集群配置]({{< relref "../configuration.md" >}}) — 通过 API 动态调整日志级别

