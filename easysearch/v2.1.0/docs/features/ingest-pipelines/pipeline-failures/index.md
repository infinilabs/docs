---
title: "处理管道故障"
date: 0001-01-01
summary: "处理管道故障 #  摄取管道由一系列按顺序执行的处理器组成。如果某个处理器失败，默认行为是整条管道中止，文档不会被索引。
你有两种方式应对处理器失败：
 中止管道（默认）：处理器失败后整条管道停止，文档不被索引 跳过失败继续执行：通过 ignore_failure 或 on_failure 让管道在失败后继续运行  默认情况下，如果管道中的某个处理器失败，则摄取管道会停止。如果您希望在处理器失败时继续运行管道，您可以在创建管道时将该处理器的 ignore_failure 参数设置为 true ：
PUT _ingest/pipeline/my-pipeline/ { &#34;description&#34;: &#34;Rename &#39;provider&#39; field to &#39;cloud.provider&#39;&#34;, &#34;processors&#34;: [ { &#34;rename&#34;: { &#34;field&#34;: &#34;provider&#34;, &#34;target_field&#34;: &#34;cloud.provider&#34;, &#34;ignore_failure&#34;: true } } ] } 您可以将 on_failure 参数指定为在处理器失败后立即运行。如果您已指定 on_failure ，即使 on_failure 配置为空，Easysearch 也会运行管道中的其他处理器：
PUT _ingest/pipeline/my-pipeline/ { &#34;description&#34;: &#34;Add timestamp to the document&#34;, &#34;processors&#34;: [ { &#34;date&#34;: { &#34;field&#34;: &#34;timestamp_field&#34;, &#34;formats&#34;: [&#34;yyyy-MM-dd HH:mm:ss&#34;], &#34;target_field&#34;: &#34;@timestamp&#34;, &#34;on_failure&#34;: [ { &#34;set&#34;: { &#34;field&#34;: &#34;ingest_error&#34;, &#34;value&#34;: &#34;failed&#34; } } ] } } ] } 如果处理器失败，Easysearch 会记录失败并继续运行搜索管道中剩余的所有处理器。要检查是否有任何失败，您可以使用摄取管道指标。"
---


# 处理管道故障

摄取管道由一系列按顺序执行的处理器组成。如果某个处理器失败，默认行为是整条管道中止，文档不会被索引。

你有两种方式应对处理器失败：

- **中止管道**（默认）：处理器失败后整条管道停止，文档不被索引
- **跳过失败继续执行**：通过 `ignore_failure` 或 `on_failure` 让管道在失败后继续运行

默认情况下，如果管道中的某个处理器失败，则摄取管道会停止。如果您希望在处理器失败时继续运行管道，您可以在创建管道时将该处理器的 `ignore_failure` 参数设置为 `true` ：

```
PUT _ingest/pipeline/my-pipeline/
{
  "description": "Rename 'provider' field to 'cloud.provider'",
  "processors": [
    {
      "rename": {
        "field": "provider",
        "target_field": "cloud.provider",
        "ignore_failure": true
      }
    }
  ]
}
```

您可以将 `on_failure` 参数指定为在处理器失败后立即运行。如果您已指定 `on_failure` ，即使 `on_failure` 配置为空，Easysearch 也会运行管道中的其他处理器：

```
PUT _ingest/pipeline/my-pipeline/
{
  "description": "Add timestamp to the document",
  "processors": [
    {
      "date": {
        "field": "timestamp_field",
        "formats": ["yyyy-MM-dd HH:mm:ss"],
        "target_field": "@timestamp",
        "on_failure": [
          {
            "set": {
              "field": "ingest_error",
              "value": "failed"
            }
          }
        ]
      }
    }
  ]
}
```

如果处理器失败，Easysearch 会记录失败并继续运行搜索管道中剩余的所有处理器。要检查是否有任何失败，您可以使用摄取管道指标。

## 摄取管道的监控指标

要查看摄取管道的监控指标，请使用节点统计 API：

```
GET /_nodes/stats/ingest?filter_path=nodes.*.ingest

```

包含所有摄取管道的统计信息，例如：

```
 {
  "nodes": {
    "iFPgpdjPQ-uzTdyPLwQVnQ": {
      "ingest": {
        "total": {
          "count": 28,
          "time_in_millis": 82,
          "current": 0,
          "failed": 9
        },
        "pipelines": {
          "user-behavior": {
            "count": 5,
            "time_in_millis": 0,
            "current": 0,
            "failed": 0,
            "processors": [
              {
                "append": {
                  "type": "append",
                  "stats": {
                    "count": 5,
                    "time_in_millis": 0,
                    "current": 0,
                    "failed": 0
                  }
                }
              }
            ]
          },
           "remove_ip": {
            "count": 5,
            "time_in_millis": 9,
            "current": 0,
            "failed": 2,
            "processors": [
              {
                "remove": {
                  "type": "remove",
                  "stats": {
                    "count": 5,
                    "time_in_millis": 8,
                    "current": 0,
                    "failed": 2
                  }
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

> 故障排除：处理摄取管道失败：您首先应该检查日志，看是否有任何错误或警告可以帮助您确定失败的原因。Easysearch 日志包含有关失败的摄取管道的信息，包括失败的处理器和失败的原因。

