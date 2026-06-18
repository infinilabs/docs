---
title: "CAT 接口"
date: 0001-01-01
description: "以表格格式查看集群、节点、索引、分片等信息。"
summary: "cat API #  您可以使用紧凑且对齐的文本 （CAT） API 以易于理解的表格格式获取有关集群的基本统计信息。cat API 是一个人类可读的接口，它返回纯文本而不是传统的 JSON。
使用 cat API，您可以回答诸如哪个节点是选定的主节点、集群处于什么状态、每个索引中有多少文档等问题。
要查看 cat API 中的可用操作，请使用以下命令：
GET _cat 还可以在查询中使用以下字符串参数。
   参数 描述     ?v 通过向列添加标题使输出更详细。它还添加了一些格式，以帮助将每列对齐在一起。此页面上的所有示例都包含 v 参数。   ?help 列出给定操作的默认标头和其他可用标头。   ?h 将输出限制为特定标头。   ?format 以 JSON、YAML 或 CBOR 格式输出结果。   ?sort 按指定列对输出进行排序。    要查看每列表示的内容，请使用 ?v 参数：
GET _cat/&lt;operation_name&gt;?v 要查看所有可用的标头，请使用 ?help 参数：
GET _cat/&lt;operation_name&gt;?help 要将输出限制为标头的子集，请使用 ?h 参数："
---


# cat API

您可以使用紧凑且对齐的文本 （CAT） API 以易于理解的表格格式获取有关集群的基本统计信息。cat API 是一个人类可读的接口，它返回纯文本而不是传统的 JSON。

使用 cat API，您可以回答诸如哪个节点是选定的主节点、集群处于什么状态、每个索引中有多少文档等问题。

要查看 cat API 中的可用操作，请使用以下命令：

```
GET _cat
```

还可以在查询中使用以下字符串参数。

| 参数      | 描述                                                                                                          |
| :-------- | :------------------------------------------------------------------------------------------------------------ |
| `?v`      | 通过向列添加标题使输出更详细。它还添加了一些格式，以帮助将每列对齐在一起。此页面上的所有示例都包含 `v` 参数。 |
| `?help`   | 列出给定操作的默认标头和其他可用标头。                                                                        |
| `?h`      | 将输出限制为特定标头。                                                                                        |
| `?format` | 以 JSON、YAML 或 CBOR 格式输出结果。                                                                          |
| `?sort`   | 按指定列对输出进行排序。                                                                                      |

要查看每列表示的内容，请使用 `?v` 参数：

```
GET _cat/<operation_name>?v
```

要查看所有可用的标头，请使用 `?help` 参数：

```
GET _cat/<operation_name>?help
```

要将输出限制为标头的子集，请使用 `?h` 参数：

```
GET _cat/<operation_name>?h=<header_name_1>,<header_name_2>&v
```

通常，对于任何操作，您可以使用 `?help` 参数找出可用的标头，然后使用 `?h` 参数将输出限制为您关注的标头。
通常，您可以使用 `?help` 参数查看任何操作可用的表头，然后使用 `?h` 参数将输出限制为您关心的表头。

## 别名 Aliases

列出别名到索引的映射，以及路由和筛选信息。

```
GET _cat/aliases?v
```

若要将信息限制为特定别名，请在查询后添加别名。

```
GET _cat/aliases/<alias>?v
```

## 分配 Allocation

列出索引的磁盘空间分配以及每个节点上的分片数。
Default request:

```
GET _cat/allocation?v
```

## 文档数 Count

列出群集中的文档数。

```
GET _cat/count?v
```

若要查看特定索引中的文档数，请在查询后添加索引名称。

```
GET _cat/count/<index>?v
```

## 字段数据 Field data

列出每个节点的每个字段使用的内存大小。

```
GET _cat/fielddata?v
```

若要将信息限制为特定字段，请在查询后添加字段名称。

```
GET _cat/fielddata/<fields>?v
```

## 集群健康状态 Health

列出群集的状态、群集已启动的时间、节点数以及有助于分析群集运行状况的其他有用信息。

```
GET _cat/health?v
```

## 索引信息 Indices

列出与索引相关的信息 - 索引使用的磁盘空间量、分片数、运行状况等。

```
GET _cat/indices?v
```

若要将信息限制为特定索引，请在查询后添加索引名称。

```
GET _cat/indices/<index>?v
```

## 主节点 Master

列出有助于识别选定主节点的信息。

```
GET _cat/master?v
```

## 节点属性 Node attributes

列出自定义节点的属性。

```
GET _cat/nodeattrs?v
```

## 节点信息 Nodes

列出节点级别信息，包括节点角色和负载指标。

一些重要的节点指标是 `pid` ， `name` ， `master` ， `ip` ， `port` ， `version` ， `build` ， `jdk` ，以及 `disk` ， `heap` ， `ram` 和 `file_desc` 。

```
GET _cat/nodes?v
```

## 待处理任务 Pending tasks

列出所有待处理任务的进度，包括任务优先级和队列中的时间。

```
GET _cat/pending_tasks?v
```

## 已安装插件 Plugins

列出已安装插件的名称、组件和版本。

```
GET _cat/plugins?v
```

## 恢复 Recovery

列出所有已完成和正在进行的索引和分片的恢复。

```
GET _cat/recovery?v
```

若要仅查看特定索引的恢复情况，请在查询后添加索引名称。

```
GET _cat/recovery/<index>?v
```

## 存储库 Repositories

列出所有快照存储库及其类型。

```
GET _cat/repositories?v
```

## 段信息 Segments

列出每个索引的 Lucene 段级别信息。

```
GET _cat/segments?v
```

若要仅查看有关特定索引段的信息，请在查询后添加索引名称。

```
GET _cat/segments/<index>?v
```

## 分片 Shards

列出所有主分片和副本分片的状态及其分布方式。

```
GET _cat/shards?v
```

若要仅查看特定索引的分片信息，请在查询后添加索引名称。

```
GET _cat/shards/<index>?v
```

## 快照 Snapshots

列出存储库的所有快照。

```
GET _cat/snapshots/<repository>?v
```

## 任务 Tasks

列出群集上当前运行的所有任务的进度。

```
GET _cat/tasks?v
```

## 索引模板 Templates

列出索引模板的名称、模式、订单号和版本号。

```
GET _cat/templates?v
```

## 线程池状态 Thread pool

列出每个节点上不同线程池的活动线程、排队线程和拒绝线程。

```
GET _cat/thread_pool?v
```

若要将信息限制为特定线程池，请在查询后添加线程池名称。

```
GET _cat/thread_pool/<thread_pool>?v
```

