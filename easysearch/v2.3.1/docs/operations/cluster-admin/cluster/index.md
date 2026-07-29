---
title: "集群管理"
date: 0001-01-01
description: "集群状态、节点管理、分片分配、设置管理。"
summary: "搭建集群 #  在深入研究 Easysearch 以及搜索和聚合数据之前，你首先需要创建一个 Easysearch 集群。
Easysearch 可以作为一个单节点或多节点集群运行。一般来说，配置两者的步骤是非常相似的。本页演示了如何创建和配置一个多节点集群，但只需做一些小的调整，你可以按照同样的步骤创建一个单节点集群。
要根据你的要求创建和部署一个 Easysearch 集群，重要的是要了解节点发现和集群形成是如何工作的，以及哪些设置对它们有影响。
有许多方法可以设计一个集群。下面的插图显示了一个基本架构。
这是一个四节点的集群，有一个专用的主节点，一个专用的协调节点，还有两个数据节点，这两个节点是主节点，也是用来摄取数据的。
下表提供了节点类型的简要描述。
   节点类型 描述 最佳实践     Master 管理集群的整体运作并跟踪集群的状态。这包括创建和删除索引，跟踪加入和离开集群的节点，检查集群中每个节点的健康状况（通过运行 ping 请求），并将分片分配给节点。 在三个不同区域的三个专用主节点是几乎所有生产用例的正确方法。这可以确保你的集群永远不会失去法定人数。两个节点在大部分时间都是空闲的，除非一个节点宕机或需要一些维护。   Data 存储和搜索数据。在本地分片上执行所有与数据有关的操作（索引、搜索、聚合）。这些是你的集群的工作节点，需要比其他任何节点类型更多的磁盘空间。 当你添加数据节点时，保持它们在各区之间的平衡。例如，如果你有三个区，以三的倍数添加数据节点，每个区一个。我们建议使用存储和内存重的节点。    默认情况下，每个节点是一个主节点和数据节点。决定节点的数量，分配节点类型，并为每个节点类型选择硬件，取决于你的使用情况。你必须考虑到一些因素，如你想保留数据的时间，你的文件的平均大小，你的典型工作负载（索引、搜索、聚合），你的预期性价比，你的风险容忍度，等等。
在你评估所有这些要求之后，我们建议你使用一个管理工具。要开始使用 INFINI Console，请参阅 INFINI Console 文档。
本页演示了如何处理不同的节点类型。它假设你有一个类似于前面插图的四节点集群。
前提条件 #  在你开始之前，你必须在你的所有节点上安装和配置 Easysearch。有关可用选项的信息，请参见 安装和配置。
完成后，使用 SSH 连接到每个节点，然后打开 config/easysearch.yml 文件。
你可以在这个文件中为你的集群设置所有的配置。
Step 1: 命名集群 #  为集群指定一个唯一的名字。如果你不指定集群名称，它将被默认设置为 easysearch。设置一个描述性的集群名称很重要，特别是如果你想在一个网络内运行多个集群。
要指定集群名称，请修改下面一行。
#cluster.name: my-application to"
---


# 搭建集群

在深入研究 Easysearch 以及搜索和聚合数据之前，你首先需要创建一个 Easysearch 集群。

Easysearch 可以作为一个单节点或多节点集群运行。一般来说，配置两者的步骤是非常相似的。本页演示了如何创建和配置一个多节点集群，但只需做一些小的调整，你可以按照同样的步骤创建一个单节点集群。

要根据你的要求创建和部署一个 Easysearch 集群，重要的是要了解节点发现和集群形成是如何工作的，以及哪些设置对它们有影响。

有许多方法可以设计一个集群。下面的插图显示了一个基本架构。

{{% load-img "/img/cluster.jpeg" %}}

这是一个四节点的集群，有一个专用的主节点，一个专用的协调节点，还有两个数据节点，这两个节点是主节点，也是用来摄取数据的。

下表提供了节点类型的简要描述。

| 节点类型 | 描述                                                                                                                                                         | 最佳实践                                                                                                                                                             |
| :------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Master` | 管理集群的整体运作并跟踪集群的状态。这包括创建和删除索引，跟踪加入和离开集群的节点，检查集群中每个节点的健康状况（通过运行 ping 请求），并将分片分配给节点。 | 在三个不同区域的三个专用主节点是几乎所有生产用例的正确方法。这可以确保你的集群永远不会失去法定人数。两个节点在大部分时间都是空闲的，除非一个节点宕机或需要一些维护。 |
| `Data`   | 存储和搜索数据。在本地分片上执行所有与数据有关的操作（索引、搜索、聚合）。这些是你的集群的工作节点，需要比其他任何节点类型更多的磁盘空间。                   | 当你添加数据节点时，保持它们在各区之间的平衡。例如，如果你有三个区，以三的倍数添加数据节点，每个区一个。我们建议使用存储和内存重的节点。                             |

默认情况下，每个节点是一个主节点和数据节点。决定节点的数量，分配节点类型，并为每个节点类型选择硬件，取决于你的使用情况。你必须考虑到一些因素，如你想保留数据的时间，你的文件的平均大小，你的典型工作负载（索引、搜索、聚合），你的预期性价比，你的风险容忍度，等等。

在你评估所有这些要求之后，我们建议你使用一个管理工具。要开始使用 INFINI Console，请参阅 [INFINI Console 文档](https://infinilabs.com/docs/latest/console/)。

本页演示了如何处理不同的节点类型。它假设你有一个类似于前面插图的四节点集群。

## 前提条件

在你开始之前，你必须在你的所有节点上安装和配置 Easysearch。有关可用选项的信息，请参见[安装和配置](https://docs.infinilabs.com/easysearch/main/docs/getting-started/install/)。

完成后，使用 SSH 连接到每个节点，然后打开 `config/easysearch.yml` 文件。

你可以在这个文件中为你的集群设置所有的配置。

## Step 1: 命名集群

为集群指定一个唯一的名字。如果你不指定集群名称，它将被默认设置为 `easysearch`。设置一个描述性的集群名称很重要，特别是如果你想在一个网络内运行多个集群。

要指定集群名称，请修改下面一行。

```yml
#cluster.name: my-application
```

to

```yml
cluster.name: easy-cluster
```

在所有的节点上做同样的修改，以确保它们会加入形成一个集群。

## Step 2: 为集群中的每个节点设置节点属性

在你命名集群后，为集群中的每个节点设置节点属性。

#### Master node

给你的主节点一个名字。如果你不指定一个名字，Easysearch 会分配一个机器生成的名字，这使得节点难以监控和排除故障。

```yml
node.name: easy-master
```

你也可以明确地指定这个节点是一个主节点。使用 `node.roles` 进行配置，推荐方式如下：

```yml
node.roles: [master]
```

这样就将该节点配置为专用的主节点，不会作为数据节点执行双重任务。

#### Data nodes

将两个节点的名称分别改为 `easy-d1` 和 `easy-d2` 。

```yml
node.name: easy-d1
```

```yml
node.name: easy-d2
```

你可以让它们既成为主节点，也作为数据节点。使用 `node.roles` 配置：

```yml
node.roles: [master, data]
```

## Step 3: 将集群与特定的 IP 地址绑定

`network_host` 定义了用于绑定节点的 IP 地址。默认情况下，Easysearch 在本地主机上监听，这将集群限制在一个节点上。你也可以使用 `_local_` 和 `_site_` 来绑定任何环回或站点本地地址，无论是 IPv4 还是 IPv6。

```yml
network.host: [_local_, _site_]
```

要形成一个多节点的集群，指定节点的 IP 地址。

```yml
network.host: <IP address of the node>
```

请确保在你的所有节点上配置这些设置。

## Step 4: 为集群配置发现主机

现在你已经配置了网络主机，你需要配置发现主机。

Zen Discovery 是内置的、默认的机制，使用[单播](https://en.wikipedia.org/wiki/Unicast)来寻找集群中的其他节点。

一般来说，你可以直接将所有符合主控条件的节点添加到 `discovery.seed_hosts` 数组中。当一个节点启动时，它会找到其他符合主控条件的节点，确定哪一个是主控，并要求加入集群。

例如，对于 `easy-master` ，这一行看起来是这样的。

```yml
discovery.seed_hosts: ["<private IP of easy-d1>", "<private IP of easy-d2>", "<private IP of easy-c1>"]
```

## Step 5: 启动集群

设置好配置后，在所有节点上启动 Easysearch。

```bash
sudo systemctl start easysearch.service
```

请参阅[以服务的形式运行 Easysearch](https://docs.infinilabs.com/easysearch/main/docs/getting-started/install/linux/#%E5%B0%86-easysearch-%E9%85%8D%E7%BD%AE%E4%B8%BA%E6%9C%8D%E5%8A%A1) ，了解如何创建和启动服务。

然后去看日志文件，看看集群的形成情况。

```bash
less /var/log/easysearch/easy-cluster.log
```

在任何节点上执行以下 `_cat` 查询，以查看作为集群的所有节点。

```bash
curl -XGET https://<private-ip>:9200/_cat/nodes?v -u 'admin:xxxxxxxxxxxx' --insecure
```

```
ip             heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
x.x.x.x           13          61   0    0.02    0.04     0.05 mi        *      easy-master
x.x.x.x           16          60   0    0.06    0.05     0.05 md        -      easy-d1
x.x.x.x           34          38   0    0.12    0.07     0.06 md        -      easy-d2
x.x.x.x           23          38   0    0.12    0.07     0.06 md        -      easy-c1
```

为了更好地了解和监控你的集群，使用[cat API](./cat.md)。

## (高级) 第 6 步：配置分片分配意识或强制意识

如果你的节点分布在几个地理区域，你可以配置分片分配意识，将所有的复制分片分配到一个与主分片不同的区域。

有了分片分配意识，如果你的一个区域的节点发生故障，你可以保证你的复制分片分布在你的其他区域。它增加了一个容错层，以确保你的数据在区域故障中幸存下来，而不仅仅是单个节点的故障。

要配置碎片分配意识，请分别向 `easy-d1` 和 `easy-d2` 添加区域属性。

```yml
node.attr.zone: zoneA
```

```yml
node.attr.zone: zoneB
```

更新集群设置。

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "zone"
  }
}
```

你可以使用 `persistent` 或 `transient` 设置。我们推荐使用 `persistent` 设置，因为它在集群重启后仍然有效。瞬时设置不会在集群重启时持续存在。

碎片分配意识试图在多个区中分离主碎片和复制碎片。但是，如果只有一个区是可用的（比如在一个区发生故障后），Easysearch 会将复制分片分配到唯一剩下的区。

另一个选择是要求主副 shards 永远不被分配到同一个区。这被称为强制意识。

要配置强制意识，为你的区属性指定所有可能的值。

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "zone",
    "cluster.routing.allocation.awareness.force.zone.values":["zoneA", "zoneB"]
  }
}
```

现在，如果一个数据节点发生故障，强制意识不会将副本分配给同一区域的节点。相反，集群进入黄色状态，只有当另一个区的节点上线时才会分配副本。

在我们的两区架构中，如果 `easy-d1` 和 `easy-d2` 的利用率低于 50% ，我们就可以使用分配意识，这样他们每个人都有存储容量来分配同一区的复制。
如果不是这样，并且 `easy-d1` 和 `easy-d2` 没有容量来容纳所有的主分片和复制分片，我们可以使用强制意识。这种方法有助于确保在发生故障时，Easysearch 不会因为缺乏存储而使你最后剩下的区域超载并锁定你的集群。

选择分配意识或强制意识取决于你在每个区可能需要多少空间来平衡你的主副分片。

## (高级）第 7 步：建立一个 hot-warm 架构

你可以设计一个 hot-warm 架构，首先将你的数据索引到热节点上--快速而昂贵--然后在一段时间后将它们转移到暖节点上--慢速而廉价。

如果你分析的是很少更新的时间序列数据，并希望将较旧的数据转移到较便宜的存储上，这种架构就很适合。

这种架构有助于节省存储成本。与其增加热节点的数量并使用快速、昂贵的存储，你可以为你不经常访问的数据增加暖节点。

要配置一个 hot-warm 的存储架构，分别给 `easy-d1` 和 `easy-d2` 添加 `temp` 属性。

```yml
node.attr.temp: hot
```

```yml
node.attr.temp: warm
```

你可以将属性名称和值设置为任何你想要的东西，只要它对你所有的热节点和温节点是一致的。

要给热节点添加一个索引 `newindex` 。

```json
PUT newindex
{
  "settings": {
    "index.routing.allocation.require.temp": "hot"
  }
}
```

请看下面的 `newindex` 的分片分配。

```json
GET _cat/shards/newindex?v
index     shard prirep state      docs store ip         node
new_index 2     p      STARTED       0  230b 10.0.0.225 easy-d1
new_index 2     r      UNASSIGNED
new_index 3     p      STARTED       0  230b 10.0.0.225 easy-d1
new_index 3     r      UNASSIGNED
new_index 4     p      STARTED       0  230b 10.0.0.225 easy-d1
new_index 4     r      UNASSIGNED
new_index 1     p      STARTED       0  230b 10.0.0.225 easy-d1
new_index 1     r      UNASSIGNED
new_index 0     p      STARTED       0  230b 10.0.0.225 easy-d1
new_index 0     r      UNASSIGNED
```

在这个例子中，所有的主碎片都被分配到 `easy-d1` ，这是我们的热节点。所有的副本碎片都没有分配，因为我们强迫这个索引只分配给热节点。

要将索引 `oldindex` 添加到热节点上。

```json
PUT oldindex
{
  "settings": {
    "index.routing.allocation.require.temp": "warm"
  }
}
```

`oldindex` 的分片分配：

```json
GET _cat/shards/oldindex?v
index     shard prirep state      docs store ip        node
old_index 2     p      STARTED       0  230b 10.0.0.74 easy-d2
old_index 2     r      UNASSIGNED
old_index 3     p      STARTED       0  230b 10.0.0.74 easy-d2
old_index 3     r      UNASSIGNED
old_index 4     p      STARTED       0  230b 10.0.0.74 easy-d2
old_index 4     r      UNASSIGNED
old_index 1     p      STARTED       0  230b 10.0.0.74 easy-d2
old_index 1     r      UNASSIGNED
old_index 0     p      STARTED       0  230b 10.0.0.74 easy-d2
old_index 0     r      UNASSIGNED
```

在这种情况下，所有的主分片都被分配到 `easy-d2` 。同样地，所有的复制分片都没有被分配，因为我们只有一个温暖的节点。

一个流行的方法是配置你的[index templates](../data-management/index-templates.md)，将 `index.routing.allocation.require.temp` 值设置为 `hot` 。这样，Easysearch 将你的最新数据存储在热节点上。

