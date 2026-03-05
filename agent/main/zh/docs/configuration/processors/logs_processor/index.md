---
title: "logs_processor"
date: 0001-01-01
summary: "logs_processor #  描述 #  logs_processor 处理器可采集任意日志。
配置示例 #  以 elasticsearch 日志为例。
pipeline: - name: log_collect auto_start: true keep_running: true retry_delay_in_ms: 3000 processor: - logs_processor: queue_name: &#34;logs&#34; logs_path: &#34;/opt/es/elasticsearch-7.7.1/logs&#34; # metadata for all log items metadata: category: elasticsearch patterns: - pattern: &#34;.*_server.json$&#34; type: json metadata: name: server timestamp_fields: [&#34;timestamp&#34;, &#34;@timestamp&#34;] remove_fields: [ &#34;type&#34;, &#34;cluster.name&#34;, &#34;cluster.uuid&#34;, &#34;node.name&#34;, &#34;node.id&#34;, &#34;timestamp&#34;, &#34;@timestamp&#34;, ] - pattern: &#34;gc.log$&#34; type: text metadata: name: gc timestamp_patterns: - &#34;\\d{4}-\\d{1,2}-\\d{1,2}T\\d{1,2}:\\d{1,2}:\\d{1,2}."
---


# logs_processor

## 描述

logs_processor 处理器可采集任意日志。

## 配置示例

以 elasticsearch 日志为例。

```yaml
pipeline:
  - name: log_collect
    auto_start: true
    keep_running: true
    retry_delay_in_ms: 3000
    processor:
      - logs_processor:
          queue_name: "logs"
          logs_path: "/opt/es/elasticsearch-7.7.1/logs"
          # metadata for all log items
          metadata:
            category: elasticsearch
          patterns:
            - pattern: ".*_server.json$"
              type: json
              metadata:
                name: server
              timestamp_fields: ["timestamp", "@timestamp"]
              remove_fields:
                [
                  "type",
                  "cluster.name",
                  "cluster.uuid",
                  "node.name",
                  "node.id",
                  "timestamp",
                  "@timestamp",
                ]
            - pattern: "gc.log$"
              type: text
              metadata:
                name: gc
              timestamp_patterns:
                - "\\d{4}-\\d{1,2}-\\d{1,2}T\\d{1,2}:\\d{1,2}:\\d{1,2}.\\d{3}\\+\\d{4}"
                - "\\d{4}-\\d{1,2}-\\d{1,2} \\d{1,2}:\\d{1,2}:\\d{1,2},\\d{3}"
                - "\\d{4}-\\d{1,2}-\\d{1,2}T\\d{1,2}:\\d{1,2}:\\d{1,2},\\d{3}"
            - pattern: ".*.log$"
              type: multiline
              line_pattern: '^\['
              metadata:
                name: server
              timestamp_patterns:
                - "\\d{4}-\\d{1,2}-\\d{1,2}T\\d{1,2}:\\d{1,2}:\\d{1,2}.\\d{3}\\+\\d{4}"
                - "\\d{4}-\\d{1,2}-\\d{1,2} \\d{1,2}:\\d{1,2}:\\d{1,2},\\d{3}"
                - "\\d{4}-\\d{1,2}-\\d{1,2}T\\d{1,2}:\\d{1,2}:\\d{1,2},\\d{3}"
```

## 参数说明

| 名称 | 类型 | 说明 |
| --- | --- | --- |
| queue_name | string | 日志采集队列名称 |
| logs_path | string | 日志路径 |
| metadata | map | 指定日志的相关元数据 |
| patterns | object | 具体日志文件的相关匹配配置，按顺序执行 |
| patterns.pattern | string | 日志文件匹配规则 |
| patterns.metadata | map | 日志文件相关元数据配置 |
| patterns.type | string | 日志类型，支持 `json`，`text`，`multiline` |
| patterns.line_pattern | string | 日志类型为 multiline 时，对于新的一行的匹配规则 |
| patterns.remove_fields | []string | 日志文件中需要移除的字段（当日志类型为 json 时可用） |
| patterns.timestamp_fields | []string | 日志数据中时间戳的字段名（当日志类型为 json 时可用） |
| patterns.timestamp_patterns | []string | 日志数据中时间戳的匹配规则（当日志类型为 text，multiline 时可用） |

