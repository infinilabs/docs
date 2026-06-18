---
title: "从数据库同步数据（JDBC / ETL）"
date: 0001-01-01
description: "从关系型数据库批量或增量同步数据到 Easysearch 的常见模式。"
summary: "从数据库同步数据（JDBC / ETL） #  将关系型数据库（MySQL、PostgreSQL 等）中的数据同步到 Easysearch 是一个常见需求。本文介绍全量导入和增量同步的主要方案和实践。
相关指南（先读这些） #    Logstash 接入  SeaTunnel 集成  Bulk API  同步方案概览 #     方案 全量 增量 实时性 复杂度 说明     Logstash JDBC Input ✅ ✅ 分钟级 低 定时轮询数据库，适合中小规模   SeaTunnel ✅ ✅ 分钟级 中 分布式 ETL，适合大数据量   Canal / Debezium (CDC) ❌ ✅ 秒级 高 基于 binlog，实时捕获变更   自研同步程序 ✅ ✅ 灵活 高 完全自定义，适合特殊需求    Logstash JDBC Input（推荐入门） #  基本配置 #  # logstash-jdbc."
---


# 从数据库同步数据（JDBC / ETL）

将关系型数据库（MySQL、PostgreSQL 等）中的数据同步到 Easysearch 是一个常见需求。本文介绍全量导入和增量同步的主要方案和实践。

## 相关指南（先读这些）

- [Logstash 接入]({{< relref "./logstash.md" >}})
- [SeaTunnel 集成]({{< relref "../third-party/seatunnel.md" >}})
- [Bulk API]({{< relref "../../features/document-operations/bulk-api.md" >}})

## 同步方案概览

| 方案                   | 全量 | 增量 | 实时性 | 复杂度 | 说明                                    |
| ---------------------- | ---- | ---- | ------ | ------ | --------------------------------------- |
| Logstash JDBC Input    | ✅   | ✅   | 分钟级 | 低     | 定时轮询数据库，适合中小规模             |
| SeaTunnel              | ✅   | ✅   | 分钟级 | 中     | 分布式 ETL，适合大数据量                 |
| Canal / Debezium (CDC) | ❌   | ✅   | 秒级   | 高     | 基于 binlog，实时捕获变更                |
| 自研同步程序           | ✅   | ✅   | 灵活   | 高     | 完全自定义，适合特殊需求                 |

## Logstash JDBC Input（推荐入门）

### 基本配置

```ruby
# logstash-jdbc.conf
input {
  jdbc {
    jdbc_driver_library => "/path/to/mysql-connector-java.jar"
    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://db-host:3306/mydb"
    jdbc_user => "reader"
    jdbc_password => "password"

    # 增量同步：记录上次的最大 update_time
    use_column_value => true
    tracking_column => "update_time"
    tracking_column_type => "timestamp"
    last_run_metadata_path => "/tmp/.logstash_jdbc_last_run"

    # 每 5 分钟执行一次
    schedule => "*/5 * * * *"

    statement => "SELECT id, title, content, category, update_time
                  FROM articles
                  WHERE update_time > :sql_last_value
                  ORDER BY update_time ASC"
  }
}

filter {
  mutate {
    copy => { "id" => "[@metadata][_id]" }
    remove_field => ["@version", "@timestamp"]
  }
}

output {
  elasticsearch {
    hosts => ["https://easysearch-node1:9200"]
    user => "admin"
    password => "your_password"
    ssl => true
    ssl_certificate_verification => false
    index => "articles"
    document_id => "%{[@metadata][_id]}"
    action => "index"
  }
}
```

### 关键参数说明

| 参数                     | 说明                                                  |
| ------------------------ | ----------------------------------------------------- |
| `tracking_column`        | 增量同步的追踪列，通常是 `update_time` 或自增 `id`    |
| `schedule`               | Cron 表达式，控制轮询频率                              |
| `last_run_metadata_path` | 记录上次同步位置的文件路径                              |
| `clean_run`              | 设为 `true` 可忽略上次记录，重新全量同步               |

## 增量同步策略

### 基于时间戳

适合有 `update_time` 或 `modified_at` 列的表：

```sql
WHERE update_time > :sql_last_value
ORDER BY update_time ASC
```

> **注意**：需要确保 `update_time` 在记录更新时会被自动刷新。

### 基于自增 ID

适合只有新增、没有修改的场景（如日志表）：

```sql
WHERE id > :sql_last_value
ORDER BY id ASC
```

### 处理删除操作

上述两种方式都无法感知已删除的记录。常见处理方式：

| 方案             | 说明                                                       |
| ---------------- | ---------------------------------------------------------- |
| 软删除标记       | 数据库中用 `is_deleted` 字段标记，同步时在 Easysearch 中过滤 |
| 全量覆盖         | 定期全量重建索引，替换旧索引（适合数据量不大的场景）        |
| CDC 方案         | 使用 Canal/Debezium 捕获 DELETE 事件，实时同步删除          |

## Mapping 设计建议

同步前应提前创建好索引和映射，避免依赖动态映射产生不理想的字段类型：

```
PUT articles
{
  "mappings": {
    "properties": {
      "id":          { "type": "keyword" },
      "title":       { "type": "text", "analyzer": "ik_max_word", "search_analyzer": "ik_smart" },
      "content":     { "type": "text", "analyzer": "ik_max_word" },
      "category":    { "type": "keyword" },
      "update_time": { "type": "date", "format": "yyyy-MM-dd HH:mm:ss||epoch_millis" }
    }
  }
}
```

## 性能优化

- **批量大小**：Logstash 默认每次读取 100000 行，可通过 `jdbc_fetch_size` 调整
- **并行写入**：配置 Logstash output 的 `workers` 数量来提升写入吞吐
- **索引刷新**：大批量导入时将 `refresh_interval` 设为 `-1`，完成后恢复
- **副本数量**：全量导入时可临时将 `number_of_replicas` 设为 `0`，完成后恢复


