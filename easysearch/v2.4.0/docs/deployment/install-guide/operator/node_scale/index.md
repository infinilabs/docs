---
title: "节点扩容"
date: 0001-01-01
summary: "节点扩容 #  Easysearch Operator 支持通过修改 YAML 配置实现快速水平扩容。
操作步骤 #  修改 Operator YAML 文件中的 replicas 字段值。例如，将集群从 3 节点扩容到 5 节点：
# 修改前 replicas: 3 # 修改后 replicas: 5 应用修改：
kubectl apply -f easysearch-cluster.yaml Operator 会并发创建新的节点（如 threenodes-masters-3、threenodes-masters-4），新节点启动后自动加入集群并参与分片分配。
 注意：扩容前请确保 Kubernetes 集群有足够的计算和存储资源。缩容操作需要谨慎，建议先手动迁移分片后再减少副本数。
 操作演示 #    "
---


# 节点扩容

Easysearch Operator 支持通过修改 YAML 配置实现快速水平扩容。

## 操作步骤

修改 Operator YAML 文件中的 `replicas` 字段值。例如，将集群从 3 节点扩容到 5 节点：

```yaml
# 修改前
replicas: 3

# 修改后
replicas: 5
```

应用修改：

```bash
kubectl apply -f easysearch-cluster.yaml
```

Operator 会**并发创建**新的节点（如 `threenodes-masters-3`、`threenodes-masters-4`），新节点启动后自动加入集群并参与分片分配。

> **注意**：扩容前请确保 Kubernetes 集群有足够的计算和存储资源。缩容操作需要谨慎，建议先手动迁移分片后再减少副本数。

## 操作演示

{{< asciinema key="/node_scale" autoplay="1" speed="2" rows="30" preload="1" >}}

