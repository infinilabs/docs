---
title: "SeaTunnel 集成"
date: 0001-01-01
description: "SeaTunnel 数据同步与 Easysearch 集成。"
summary: "SeaTunnel 集成 #   Apache SeaTunnel 是一款高性能分布式数据集成平台，支持海量数据的实时和离线同步。
SeaTunnel 从 2.3.4 版本开始内置了原生的 INFINI Easysearch Connector，同时支持 Source（读取）和 Sink（写入），使用 easysearch-client 专用客户端，无需依赖 Elasticsearch 兼容层。
 Sink 关键特性：Exactly-Once 语义、CDC（INSERT / UPDATE / DELETE）、HTTPS/TLS。 Source 关键特性：Batch / Stream 模式、Exactly-Once、列投影、并行读取、自定义分片。
 典型场景 #     场景 Source → Sink     MySQL 全量/增量同步 MySQL → Easysearch   日志接入 Kafka → Easysearch   数据仓库导出 Hive / ClickHouse → Easysearch   跨集群迁移 Elasticsearch → Easysearch   Easysearch 数据导出 Easysearch → ClickHouse / Kafka / 文件   CDC 实时同步 MySQL CDC → Easysearch    安装 SeaTunnel #  # 下载（建议使用 2."
---


# SeaTunnel 集成

[Apache SeaTunnel](https://seatunnel.apache.org/) 是一款高性能分布式数据集成平台，支持海量数据的实时和离线同步。

SeaTunnel **从 2.3.4 版本开始内置了原生的 INFINI Easysearch Connector**，同时支持 [Source](https://seatunnel.apache.org/docs/2.3.12/connector-v2/source/Easysearch)（读取）和 [Sink](https://seatunnel.apache.org/docs/2.3.12/connector-v2/sink/Easysearch)（写入），使用 [easysearch-client](https://central.sonatype.com/artifact/com.infinilabs/easysearch-client) 专用客户端，无需依赖 Elasticsearch 兼容层。

> **Sink 关键特性**：Exactly-Once 语义、CDC（INSERT / UPDATE / DELETE）、HTTPS/TLS。
> **Source 关键特性**：Batch / Stream 模式、Exactly-Once、列投影、并行读取、自定义分片。

## 典型场景

| 场景 | Source → Sink |
|------|--------------|
| MySQL 全量/增量同步 | MySQL → Easysearch |
| 日志接入 | Kafka → Easysearch |
| 数据仓库导出 | Hive / ClickHouse → Easysearch |
| 跨集群迁移 | Elasticsearch → Easysearch |
| Easysearch 数据导出 | Easysearch → ClickHouse / Kafka / 文件 |
| CDC 实时同步 | MySQL CDC → Easysearch |

## 安装 SeaTunnel

```bash
# 下载（建议使用 2.3.12）
wget https://archive.apache.org/dist/seatunnel/2.3.12/apache-seatunnel-2.3.12-bin.tar.gz
tar -xzf apache-seatunnel-2.3.12-bin.tar.gz
cd apache-seatunnel-2.3.12
```

## 数据类型映射

| Easysearch 类型 | SeaTunnel 类型 |
|-----------------|----------------|
| STRING / KEYWORD / TEXT | STRING |
| BOOLEAN | BOOLEAN |
| BYTE | BYTE |
| SHORT | SHORT |
| INTEGER | INT |
| LONG | LONG |
| FLOAT / HALF_FLOAT | FLOAT |
| DOUBLE | DOUBLE |
| Date | LOCAL_DATE_TIME_TYPE |

## 配置示例：MySQL → Easysearch

创建任务配置文件 `mysql-to-easysearch.conf`：

```hocon
env {
  parallelism = 2
  job.mode = "BATCH"
}

source {
  Jdbc {
    url = "jdbc:mysql://localhost:3306/mydb"
    driver = "com.mysql.cj.jdbc.Driver"
    user = "root"
    password = "password"
    query = "SELECT id, title, content, created_at FROM articles"
  }
}

transform {
}

sink {
  Easysearch {
    hosts = ["https://localhost:9200"]
    username = "admin"
    password = "your-password"
    index = "articles"
    tls_verify_certificate = false
    tls_verify_hostname = false
  }
}
```

运行：

```bash
bin/seatunnel.sh --config mysql-to-easysearch.conf
```

## 配置示例：Kafka → Easysearch（实时）

```hocon
env {
  parallelism = 4
  job.mode = "STREAMING"
  checkpoint.interval = 5000
}

source {
  Kafka {
    bootstrap.servers = "kafka:9092"
    topic = "app-logs"
    format = "json"
    consumer.group = "seatunnel-es"
    start_mode = "latest"
  }
}

sink {
  Easysearch {
    hosts = ["https://localhost:9200"]
    username = "admin"
    password = "your-password"
    index = "app-logs-${now('yyyy-MM-dd')}"
    tls_verify_certificate = false
  }
}
```

## 配置示例：ES → Easysearch 迁移

```hocon
env {
  parallelism = 4
  job.mode = "BATCH"
}

source {
  Elasticsearch {
    hosts = ["http://old-es:9200"]
    index = "old-index"
    query = {"match_all": {}}
    scroll_time = "5m"
    scroll_size = 1000
  }
}

sink {
  Easysearch {
    hosts = ["https://localhost:9200"]
    username = "admin"
    password = "your-password"
    index = "new-index"
    tls_verify_certificate = false
  }
}
```

## 配置示例：Easysearch 作为 Source 读取数据

Easysearch Connector 同时提供 Source 能力，可以从 Easysearch 读取数据同步到其他系统：

```hocon
env {
  parallelism = 4
  job.mode = "BATCH"
}

source {
  Easysearch {
    hosts = ["https://localhost:9200"]
    username = "admin"
    password = "your-password"
    index = "articles"
    source = ["_id", "title", "content", "views"]
    query = {"match_all": {}}
    scroll_time = "5m"
    scroll_size = 1000
    tls_verify_certificate = false
  }
}

sink {
  Console {}
}
```

> Source 名称同样是 **`Easysearch`**，支持通配符索引匹配（如 `seatunnel-*`）和 DSL 查询过滤。

## 配置示例：CDC 实时同步

```hocon
env {
  parallelism = 2
  job.mode = "STREAMING"
  checkpoint.interval = 5000
}

source {
  MySQL-CDC {
    hostname = "localhost"
    port = 3306
    username = "root"
    password = "password"
    database-name = "mydb"
    table-name = "orders"
  }
}

sink {
  Easysearch {
    hosts = ["https://localhost:9200"]
    username = "admin"
    password = "your-password"
    index = "orders"
    primary_keys = ["order_id"]
    tls_verify_certificate = false
  }
}
```

> CDC 模式下必须指定 `primary_keys`，Easysearch Connector 会自动将 INSERT / UPDATE / DELETE 事件映射为对应的文档操作。

## 配置示例：TLS 证书验证

```hocon
sink {
  Easysearch {
    hosts = ["https://localhost:9200"]
    username = "admin"
    password = "your-password"
    index = "my-index"
    tls_keystore_path = "/path/to/easysearch/config/certs/http.p12"
    tls_keystore_password = "your-keystore-password"
  }
}
```

## Sink 参数说明

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|:----:|--------|------|
| `hosts` | list | ✅ | — | Easysearch 节点地址，格式 `host:port` |
| `index` | string | ✅ | — | 目标索引名，支持字段变量如 `seatunnel_${age}` |
| `primary_keys` | list | — | — | 用于生成 `_id` 的主键字段（CDC 模式必填） |
| `key_delimiter` | string | — | `_` | 复合主键的分隔符 |
| `username` | string | — | — | 认证用户名 |
| `password` | string | — | — | 认证密码 |
| `max_batch_size` | int | — | 10 | 批量写入的文档数 |
| `max_retry_count` | int | — | 3 | 失败重试次数 |
| `tls_verify_certificate` | bool | — | `true` | 是否验证 TLS 证书 |
| `tls_verify_hostname` | bool | — | `true` | 是否验证主机名 |
| `tls_keystore_path` | string | — | — | PEM 或 JKS 密钥库路径 |
| `tls_keystore_password` | string | — | — | 密钥库密码 |
| `tls_truststore_path` | string | — | — | PEM 或 JKS 信任库路径 |
| `tls_truststore_password` | string | — | — | 信任库密码 |
| `schema_save_mode` | enum | — | `CREATE_SCHEMA_WHEN_NOT_EXIST` | 目标索引处理策略：`RECREATE_SCHEMA` / `CREATE_SCHEMA_WHEN_NOT_EXIST` / `ERROR_WHEN_SCHEMA_NOT_EXIST` / `IGNORE` |
| `data_save_mode` | enum | — | `APPEND_DATA` | 目标数据处理策略：`DROP_DATA` / `APPEND_DATA` / `ERROR_WHEN_DATA_EXISTS` |

## Source 参数说明

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|:----:|--------|------|
| `hosts` | list | ✅ | — | Easysearch 节点地址，格式 `host:port` |
| `index` | string | ✅ | — | 源索引名，支持 `*` 通配符匹配 |
| `username` | string | — | — | 认证用户名 |
| `password` | string | — | — | 认证密码 |
| `source` | list | — | — | 要读取的字段列表，可含 `_id`；不配置则需配置 `schema` |
| `query` | json | — | — | Easysearch DSL 查询条件，控制读取范围 |
| `scroll_time` | string | — | — | Scroll 上下文保持时间 |
| `scroll_size` | int | — | — | 每次 Scroll 请求返回的最大文档数 |
| `schema` | object | — | — | 数据结构定义；不配置则需配置 `source` |
| `tls_verify_certificate` | bool | — | `true` | 是否验证 TLS 证书 |
| `tls_verify_hostname` | bool | — | `true` | 是否验证主机名 |
| `tls_keystore_path` | string | — | — | PEM 或 JKS 密钥库路径 |
| `tls_keystore_password` | string | — | — | 密钥库密码 |
| `tls_truststore_path` | string | — | — | PEM 或 JKS 信任库路径 |
| `tls_truststore_password` | string | — | — | 信任库密码 |

## 注意事项

| 注意项 | 说明 |
|--------|------|
| Connector 名称 | Source 和 Sink 名称均为 **`Easysearch`**（不是 `Elasticsearch`），这是 SeaTunnel 的原生连接器 |
| 版本要求 | SeaTunnel ≥ 2.3.4（建议 2.3.12） |
| HTTPS | Easysearch 默认启用 HTTPS，需配置 `tls_verify_certificate = false` 或提供证书 |
| Mapping | 建议提前创建目标索引的 Mapping，避免自动映射不符合预期 |
| 性能 | 调整 `parallelism` 和 `max_batch_size` 以优化吞吐 |

## 延伸阅读

- [SeaTunnel Easysearch Sink 官方文档](https://seatunnel.apache.org/docs/2.3.12/connector-v2/sink/Easysearch)
- [SeaTunnel Easysearch Source 官方文档](https://seatunnel.apache.org/docs/2.3.12/connector-v2/source/Easysearch)
- [GitHub: connector-easysearch](https://github.com/apache/seatunnel/tree/dev/seatunnel-connectors-v2/connector-easysearch)
- [数据接入]({{< relref "/docs/integrations/ingest/" >}})
- [Bulk API]({{< relref "/docs/features/document-operations/bulk-api.md" >}})

