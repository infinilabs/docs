---
title: "日志管理"
date: 0001-01-01
description: "集群日志的查看、配置与分析。"
summary: "系统日志 #  Easysearch 日志包含监控群集操作和故障排除问题的重要信息。日志的位置因安装类型而异：
 在 Docker 上，Easysearch 将大多数日志写入控制台，并将其余日志存储在 easysearch/logs/ 中。tarball 安装也使用 easysearch/logs/。 在 RPM 和 Debian 安装上，Easysearch 将日志写入 /var/log/easysearch/。  日志可作为 .log （纯文本）和 .json 文件使用。
应用程序日志 #  对于其应用程序日志，Easysearch 使用 Apache Log4j 2 其内置日志级别（从最低到最高）为 TRACE 、 DEBUG 、 INFO 、 WARN 、 ERROR 和 FATAL 。默认 Easysearch 日志级别为 INFO 。
您可以更改各个 Easysearch 模块的日志级别，而不是更改默认日志级别（ logger.level ）：
PUT /_cluster/settings { &#34;persistent&#34; : { &#34;logger.org.easysearch.index.reindex&#34; : &#34;DEBUG&#34; } } 此示例更改后，Easysearch 在重新索引操作期间会发出更详细的日志："
---


# 系统日志

Easysearch 日志包含监控群集操作和故障排除问题的重要信息。日志的位置因安装类型而异：

- 在 Docker 上，Easysearch 将大多数日志写入控制台，并将其余日志存储在 `easysearch/logs/` 中。tarball 安装也使用 `easysearch/logs/`。
- 在 RPM 和 Debian 安装上，Easysearch 将日志写入 `/var/log/easysearch/`。

日志可作为 `.log` （纯文本）和 `.json` 文件使用。

## 应用程序日志

对于其应用程序日志，Easysearch 使用 [Apache Log4j 2](https://logging.apache.org/log4j/2.x/) 其内置日志级别（从最低到最高）为 TRACE 、 DEBUG 、 INFO 、 WARN 、 ERROR 和 FATAL 。默认 Easysearch 日志级别为 INFO 。

您可以更改各个 Easysearch 模块的日志级别，而不是更改默认日志级别（ `logger.level` ）：

```json
PUT /_cluster/settings
{
  "persistent" : {
    "logger.org.easysearch.index.reindex" : "DEBUG"
  }
}
```

此示例更改后，Easysearch 在重新索引操作期间会发出更详细的日志：

```
[2019-10-18T16:52:51,184][DEBUG][o.e.i.r.TransportReindexAction] [node1] [1626]: starting
[2019-10-18T16:52:51,186][DEBUG][o.e.i.r.TransportReindexAction] [node1] executing initial scroll against [some-index]
[2019-10-18T16:52:51,291][DEBUG][o.e.i.r.TransportReindexAction] [node1] scroll returned [3] documents with a scroll id of [DXF1Z==]
[2019-10-18T16:52:51,292][DEBUG][o.e.i.r.TransportReindexAction] [node1] [1626]: got scroll response with [3] hits
[2019-10-18T16:52:51,294][DEBUG][o.e.i.r.WorkerBulkByScrollTaskState] [node1] [1626]: preparing bulk request for [0s]
[2019-10-18T16:52:51,297][DEBUG][o.e.i.r.TransportReindexAction] [node1] [1626]: preparing bulk request
[2019-10-18T16:52:51,299][DEBUG][o.e.i.r.TransportReindexAction] [node1] [1626]: sending [3] entry, [222b] bulk request
[2019-10-18T16:52:51,310][INFO ][o.e.c.m.MetaDataMappingService] [node1] [some-new-index/R-j3adc6QTmEAEb-eAie9g] create_mapping [_doc]
[2019-10-18T16:52:51,383][DEBUG][o.e.i.r.TransportReindexAction] [node1] [1626]: got scroll response with [0] hits
[2019-10-18T16:52:51,384][DEBUG][o.e.i.r.WorkerBulkByScrollTaskState] [node1] [1626]: preparing bulk request for [0s]
[2019-10-18T16:52:51,385][DEBUG][o.e.i.r.TransportReindexAction] [node1] [1626]: preparing bulk request
[2019-10-18T16:52:51,386][DEBUG][o.e.i.r.TransportReindexAction] [node1] [1626]: finishing without any catastrophic failures
[2019-10-18T16:52:51,395][DEBUG][o.e.i.r.TransportReindexAction] [node1] Freed [1] contexts
```

DEBUG 和 TRACE 级别非常冗长。如果启用其中一个来解决问题，请在完成后禁用它。

还有其他方法可以更改日志级别：

1. 向 `easysearch.yml` 添加行：

```yml
logger.org.easysearch.index.reindex: debug
```

如果您希望在多个集群中重用日志记录配置，或者使用单个节点调试启动问题，那么修改 `easysearch.yml` 最有意义。

2. 修改 `log4j2.properties`：

```
   # 定义一个唯一 ID 为 reindex 的新 logger
   logger.reindex.name = org.easysearch.index.reindex
   # 设置该 logger 的日志级别
   logger.reindex.level = debug
```

这种方法非常灵活，但需要熟悉 [Log4j 2 属性文件语法](https://logging.apache.org/log4j/2.x/manual/configuration.html#Properties). 通常，其他选项提供了更简单的配置体验。

如果检查配置目录中的默认 `log4j2.properties` 文件，可以看到一些 Easysearch 特定的变量：

```
   appender.console.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %m%n
   appender.rolling_old.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}.log
```

- `${sys:es.logs.base_path}` 是日志目录（例如 `/var/log/easysearch/`）。
- `${sys:es.logs.cluster_name}` 是集群名称。
- `[%node_name]` 是节点名称。

## 慢日志

Easysearch 有两个 _slow logs_ ，帮助您识别性能问题的日志：搜索慢日志和索引慢日志。

这些日志依赖于阈值来定义什么是“慢”搜索或索引操作。例如，如果完成一个查询需要 15 秒以上的时间，您可能会认为它很慢。与为模块配置的应用程序日志不同，为索引配置慢日志。默认情况下，两个日志都被禁用（所有阈值都设置为“-1”）：

```json
GET <some-index>/_settings?include_defaults=true

{
  "indexing": {
    "slowlog": {
      "reformat": "true",
      "threshold": {
        "index": {
          "warn": "-1",
          "trace": "-1",
          "debug": "-1",
          "info": "-1"
        }
      },
      "source": "1000",
      "level": "TRACE"
    }
  },
  "search": {
    "slowlog": {
      "level": "TRACE",
      "threshold": {
        "fetch": {
          "warn": "-1",
          "trace": "-1",
          "debug": "-1",
          "info": "-1"
        },
        "query": {
          "warn": "-1",
          "trace": "-1",
          "debug": "-1",
          "info": "-1"
        }
      }
    }
  }
}
```

要启用这些日志，请增加一个或多个阈值：

```json
PUT <some-index>/_settings
{
  "indexing": {
    "slowlog": {
      "threshold": {
        "index": {
          "warn": "15s",
          "trace": "750ms",
          "debug": "3s",
          "info": "10s"
        }
      },
      "source": "500",
      "level": "INFO"
    }
  }
}
```

在此示例中，Easysearch 记录 WARN 级别需要 15 秒或更长时间的索引操作，以及 INFO 级别需要 10 到 14._x_ 秒的操作。如果将阈值设置为 0 秒，Easysearch 将记录所有操作，这对于测试是否确实启用了慢速日志非常有用。

- `reformat` 指定是将文档 `_source` 字段记录为单行（ `true` ）还是让它跨越多行（ `false` ）。
- `source` 是要记录的文档 `_source` 字段的字符数。
- `level` 是要包含的最低日志级别。

`easysearch_index_indexing_slowlog.log` 中的一行可能如下所示：

```
node1 | [2019-10-24T19:48:51,012][WARN][i.i.s.index] [node1] [some-index/i86iF5kyTyy-PS8zrdDeAA] took[3.4ms], took_millis[3], type[_doc], id[1], routing[], source[{"title":"Your Name", "Director":"Makoto Shinkai"}]
```

如果将阈值或级别设置得太低，慢日志可能会占用大量磁盘空间。考虑暂时启用它们以进行故障排除或性能调整。要禁用慢速日志，请将所有阈值返回到 `-1` 。

## 弃用日志

弃用日志记录客户端对集群进行弃用 API 调用的时间。这些日志可以帮助您在升级到新的主要版本之前识别并修复问题。默认情况下，Easysearch 在 WARN 级别记录不推荐的 API 调用，这几乎适用于所有用例。如果需要，可以使用 `_cluster/settings` 、 `easysearch.yml` 或 `log4j2.properties` 配置 `logger.deprecation.level` 。

