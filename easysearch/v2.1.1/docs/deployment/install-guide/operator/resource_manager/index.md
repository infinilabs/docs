---
title: "资源扩容"
date: 0001-01-01
summary: "资源扩容 #  Easysearch Operator 支持对集群的 CPU、内存和磁盘资源进行在线扩容，扩容过程采用滚动更新方式，不影响集群可用性。
查看当前资源 #  kubectl get sts/threenodes-masters -o yaml 示例输出（关键部分）：
resources: requests: cpu: &#34;1&#34; memory: 3Gi limits: cpu: &#34;1&#34; memory: 5Gi # 存储 resources: requests: storage: 30Gi 修改资源配置 #  编辑 Operator YAML 文件，调整资源限制：
resources: requests: cpu: &#34;1&#34; memory: 4Gi limits: cpu: &#34;2&#34; memory: 6Gi # 磁盘扩容 resources: requests: storage: 50Gi 应用修改：
kubectl apply -f easysearch-cluster.yaml  注意：磁盘扩容依赖于 StorageClass 是否支持在线扩容（allowVolumeExpansion: true）。
 滚动更新流程 #  Operator 会按顺序逐个更新节点："
---


# 资源扩容

Easysearch Operator 支持对集群的 CPU、内存和磁盘资源进行在线扩容，扩容过程采用**滚动更新**方式，不影响集群可用性。

## 查看当前资源

```bash
kubectl get sts/threenodes-masters -o yaml
```

示例输出（关键部分）：

```yaml
resources:
  requests:
    cpu: "1"
    memory: 3Gi
  limits:
    cpu: "1"
    memory: 5Gi

# 存储
resources:
  requests:
    storage: 30Gi
```

## 修改资源配置

编辑 Operator YAML 文件，调整资源限制：

```yaml
resources:
  requests:
    cpu: "1"
    memory: 4Gi
  limits:
    cpu: "2"
    memory: 6Gi

# 磁盘扩容
resources:
  requests:
    storage: 50Gi
```

应用修改：

```bash
kubectl apply -f easysearch-cluster.yaml
```

> **注意**：磁盘扩容依赖于 StorageClass 是否支持在线扩容（`allowVolumeExpansion: true`）。

## 滚动更新流程

Operator 会按顺序逐个更新节点：

1. 从 `threenodes-masters-0` 开始更新
2. 等待该节点完全就绪后，更新 `threenodes-masters-1`
3. 依次滚动，直到所有节点更新完毕

更新完成后可验证资源是否生效：

```bash
kubectl get sts/threenodes-masters -o yaml | grep -A5 resources
```

## 操作演示

{{< asciinema key="/resouce_manager" autoplay="1" speed="2" rows="30" preload="1" >}}

