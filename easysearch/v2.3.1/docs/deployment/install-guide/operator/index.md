---
title: "k8s Operator"
date: 0001-01-01
summary: "Easysearch Operator 概述 #  介绍 #  Easysearch Operator 为 Kubernetes 环境下的 Easysearch 集群提供了自动化管理和操作能力。该 Operator 借助 Kubernetes 的 Operator 框架 Kubebuilder，通过自定义资源定义（ CRD）以及相应的控制逻辑对 Easysearch 进行管理和操作，以实现在 Kubernetes 上管理 Easysearch 服务生命周期的需求。
Operator 将运维经验沉淀为代码，实现运维的代码化、自动化、智能化。以往的高可用、扩展收缩、故障恢复等运维操作，都通过 Operator 进行沉淀下来，将运维经验通过代码的方式进行固化和传承，减少人为故障的概率，提升整个运维的效率。与原生的资源（Pod/SVC/PV/PVC 等）一样，Operator 将应用集群也视为一种资源，借助 Kubernetes 已有的工作机制和框架更为便捷灵活地实现。
特性和优势 #     特性 说明     生命周期管理 自动化处理集群的部署、升级、扩容、备份、密码修改及恢复等复杂操作   声明式配置 用户只需声明所需的最终状态，Operator 自动确保集群的当前状态与期望状态匹配   配置简化 通过 SearchCluster CRD、ConfigMap 和 Secrets 管理集群参数和敏感信息   水平/垂直伸缩 支持节点水平扩缩容（修改 replicas）和资源垂直扩容（CPU/内存/磁盘）   滚动升级 逐节点滚动升级，升级过程中集群保持可用   自我修复 监控集群状态，出现问题时自动进行故障转移和恢复   证书自动管理 集成 cert-manager，自动颁发和续期 TLS 证书（transport/http/admin 三套）   S3 定期备份 通过 SLM API 实现自动化快照备份策略    架构原理 #  Operator 遵循 Kubernetes 控制器模式，核心工作流如下："
---


# Easysearch Operator 概述

## 介绍

**Easysearch Operator** 为 Kubernetes 环境下的 Easysearch 集群提供了自动化管理和操作能力。该 Operator 借助 Kubernetes 的 Operator [框架 Kubebuilder](https://github.com/kubernetes-sigs/kubebuilder)，通过自定义资源定义（[CRD](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/api-extension/custom-resources/)）以及相应的控制逻辑对 Easysearch 进行管理和操作，以实现在 Kubernetes 上管理 Easysearch 服务生命周期的需求。

Operator 将运维经验沉淀为代码，实现运维的代码化、自动化、智能化。以往的高可用、扩展收缩、故障恢复等运维操作，都通过 Operator 进行沉淀下来，将运维经验通过代码的方式进行固化和传承，减少人为故障的概率，提升整个运维的效率。与原生的资源（Pod/SVC/PV/PVC 等）一样，Operator 将**应用集群**也视为一种资源，借助 Kubernetes 已有的工作机制和框架更为便捷灵活地实现。

## 特性和优势

| 特性 | 说明 |
|------|------|
| **生命周期管理** | 自动化处理集群的部署、升级、扩容、备份、密码修改及恢复等复杂操作 |
| **声明式配置** | 用户只需声明所需的最终状态，Operator 自动确保集群的当前状态与期望状态匹配 |
| **配置简化** | 通过 SearchCluster CRD、ConfigMap 和 Secrets 管理集群参数和敏感信息 |
| **水平/垂直伸缩** | 支持节点水平扩缩容（修改 replicas）和资源垂直扩容（CPU/内存/磁盘） |
| **滚动升级** | 逐节点滚动升级，升级过程中集群保持可用 |
| **自我修复** | 监控集群状态，出现问题时自动进行故障转移和恢复 |
| **证书自动管理** | 集成 cert-manager，自动颁发和续期 TLS 证书（transport/http/admin 三套） |
| **S3 定期备份** | 通过 SLM API 实现自动化快照备份策略 |

## 架构原理

Operator 遵循 Kubernetes 控制器模式，核心工作流如下：

```text
┌────────────────────────────────────────────────────────────────┐
│  用户提交 SearchCluster YAML                                     │
│       ↓                                                        │
│  API Server 存储 CRD 对象                                       │
│       ↓                                                        │
│  Operator Controller (Reconcile Loop)                          │
│       │                                                        │
│       ├─→ 创建/管理 StatefulSet (Easysearch 节点)                │
│       ├─→ 创建/管理 Service (集群网络)                            │
│       ├─→ 创建/管理 PVC (持久化存储)                              │
│       ├─→ 创建/管理 ConfigMap/Secret (配置/凭证)                  │
│       ├─→ 创建/管理 Job (安全配置更新/快照任务)                    │
│       └─→ 创建/管理 PodDisruptionBudget (可用性保障)              │
│       ↓                                                        │
│  持续监控 → 发现偏差 → 自动修正 (自我修复)                        │
└────────────────────────────────────────────────────────────────┘
```

Operator Deployment 本身为无状态服务，通过 Watch 机制监听 `SearchCluster` 资源变化，自动驱动 Reconcile 循环。

## 环境要求

| 组件 | 要求 | 说明 |
|------|------|------|
| Kubernetes | ≥ 1.9 | StatefulSet 自 1.9 版本起成为管理有状态应用的标准方式 |
| StorageClass | 必须提前配置 | 建议设置 `allowVolumeExpansion: true` 以支持在线磁盘扩容 |
| cert-manager | 需要安装 | 自动化管理 Easysearch 集群 TLS 证书的颁发和续期 |
| RBAC | ServiceAccount + ClusterRole | Operator 需要操作 StatefulSet/Pod/Service/Secret/PVC/Job 等资源的权限 |
| 容器镜像 | `infinilabs/operator:latest` | 约 60MB，可从 [Docker Hub](https://hub.docker.com/r/infinilabs/operator) 获取 |
| Easysearch 镜像 | `infinilabs/easysearch` | 根据版本选择对应 tag |

## 快速入门

从零到集群运行只需以下几步：

```bash
# 1. 部署 RBAC（ServiceAccount + ClusterRole + ClusterRoleBinding）
kubectl apply -f rbac.yaml

# 2. 部署 cert-manager 并创建证书
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
kubectl apply -f certificates.yaml

# 3. 部署 Operator
kubectl apply -f operator-deployment.yaml

# 4. 部署 Easysearch 集群
kubectl apply -f easysearch-cluster.yaml

# 5. 验证集群状态
kubectl exec -it threenodes-masters-0 -- \
  curl -ku admin:PASSWORD https://localhost:9200/_cluster/health?pretty
```

> 每个步骤的详细配置文件和参数说明，请参考下方的子页面。

## CRD 字段参考

`SearchCluster` 是 Operator 定义的自定义资源（CRD），完整的 Spec 字段如下：

### `spec.security` — 安全配置

| 字段 | 类型 | 说明 |
|------|------|------|
| `config.adminSecret.name` | string | admin 管理员证书 Secret 名称 |
| `config.adminCredentialsSecret.name` | string | 管理员用户名/密码 Secret 名称 |
| `tls.http.generate` | bool | 是否自动生成 HTTP 证书（`false` 则需提供自定义证书） |
| `tls.http.secret.name` | string | HTTP 访问证书 Secret 名称 |
| `tls.transport.generate` | bool | 是否自动生成 transport 证书 |
| `tls.transport.perNode` | bool | 是否为每个节点单独配置证书 |
| `tls.transport.secret.name` | string | 节点间通信证书 Secret 名称 |
| `tls.transport.nodesDn` | []string | 节点证书 DN（Distinguished Name）列表 |
| `tls.transport.adminDn` | []string | 管理员证书 DN 列表 |

### `spec.general` — 通用配置

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `version` | string | — | Easysearch 版本号，如 `"1.7.0-223"` |
| `httpPort` | int | 9200 | HTTP 监听端口 |
| `vendor` | string | Easysearch | 固定为 `Easysearch` |
| `serviceAccount` | string | — | 关联的 K8s ServiceAccount 名称 |
| `serviceName` | string | — | Service 名称，也是集群名称 |
| `drainDataNodes` | bool | false | 删除节点前是否先迁移分片数据 |
| `monitoring.enable` | bool | false | 是否开启 Prometheus 监控（需安装 ServiceMonitor CRD） |
| `pluginsList` | []string | [] | 需要安装的 Easysearch 插件列表 |
| `snapshotRepositories` | []object | — | S3 快照仓库配置，见下表 |

#### `snapshotRepositories` 子字段

| 字段 | 说明 |
|------|------|
| `name` | 快照仓库名称 |
| `type` | 快照类型，如 `s3` |
| `settings.bucket` | S3 桶名称（需提前创建） |
| `settings.access_key` | S3 访问密钥 |
| `settings.secret_key` | S3 密钥 |
| `settings.endpoint` | S3 访问地址 |

### `spec.nodePools[]` — 节点池配置

| 字段 | 类型 | 说明 |
|------|------|------|
| `component` | string | 节点池标识名称，如 `masters`、`data`、`coordinators` |
| `replicas` | int | 节点数量 |
| `diskSize` | string | 单节点磁盘大小，如 `"30Gi"` |
| `jvm` | string | JVM 参数，如 `-Xmx4G -Xms4G` |
| `roles` | []string | 节点角色列表：`master`、`data`、`ingest`、`search` |
| `nodeSelector` | map | Kubernetes 节点选择器，用于调度亲和 |
| `resources.requests.cpu` | string | CPU 申请值 |
| `resources.requests.memory` | string | 内存申请值 |
| `resources.limits.cpu` | string | CPU 上限 |
| `resources.limits.memory` | string | 内存上限 |

## 高阶配置

### 节点角色分离

生产环境建议将 master、data、coordinating 节点分开部署，通过配置多个 `nodePools` 实现：

```yaml
nodePools:
  - component: masters
    replicas: 3
    diskSize: "10Gi"
    jvm: -Xmx2G -Xms2G
    roles:
      - "master"
    resources:
      requests:
        memory: "2Gi"
        cpu: "500m"
      limits:
        memory: "3Gi"
        cpu: "1"

  - component: data
    replicas: 5
    diskSize: "200Gi"
    jvm: -Xmx16G -Xms16G
    roles:
      - "data"
    resources:
      requests:
        memory: "16Gi"
        cpu: "2"
      limits:
        memory: "20Gi"
        cpu: "4"

  - component: coordinators
    replicas: 2
    diskSize: "10Gi"
    jvm: -Xmx4G -Xms4G
    roles:
      - "ingest"
    resources:
      requests:
        memory: "4Gi"
        cpu: "1"
      limits:
        memory: "6Gi"
        cpu: "2"
```

### 节点调度与亲和性

通过 `nodeSelector` 将 Easysearch Pod 调度到指定的 Kubernetes 节点：

```yaml
nodePools:
  - component: data
    replicas: 3
    nodeSelector:
      disktype: ssd
      node-role: easysearch-data
    # ...
```

如果需要更精细的调度策略（如反亲和性确保 Pod 分散到不同宿主机），可以通过 Kubernetes 原生的 `PodAntiAffinity` 实现。建议至少保证 master 节点分布在不同的物理节点或可用区：

```bash
# 给 K8s 节点打标签
kubectl label nodes node-1 node-role=easysearch-master
kubectl label nodes node-2 node-role=easysearch-master
kubectl label nodes node-3 node-role=easysearch-master
```

### Service 暴露方式

Operator 默认创建 ClusterIP 类型的 Service，仅集群内可访问。如需从外部访问，可手动创建其他类型的 Service：

**NodePort 方式**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: easysearch-nodeport
spec:
  type: NodePort
  selector:
    app: threenodes     # 匹配 Easysearch Pod 的 label
  ports:
    - port: 9200
      targetPort: 9200
      nodePort: 30920   # 宿主机端口，范围 30000-32767
```

**LoadBalancer 方式（云环境）**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: easysearch-lb
spec:
  type: LoadBalancer
  selector:
    app: threenodes
  ports:
    - port: 9200
      targetPort: 9200
```

**Ingress 方式**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: easysearch-ingress
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  rules:
    - host: easysearch.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: threenodes
                port:
                  number: 9200
```

### 监控集成

Operator 内置了 Prometheus ServiceMonitor 支持（ClusterRole 中已包含 `servicemonitors` 资源的操作权限）。

开启监控：

```yaml
general:
  monitoring:
    enable: true
```

启用后，Operator 会自动创建 ServiceMonitor 资源，Prometheus 可自动发现并采集 Easysearch 指标。

搭配 Grafana 使用时，可导入 Easysearch 的 Dashboard 模板来可视化集群状态、节点负载、索引性能等指标。

### 插件管理

通过 `pluginsList` 字段配置需要在 Easysearch 节点启动时安装的插件：

```yaml
general:
  pluginsList:
    - analysis-ik
    - analysis-pinyin
```

> 注意：插件安装会在 Pod 启动阶段执行，可能会增加启动时间。插件列表变更后会触发滚动更新。

### 离线部署

在无法访问公网的环境中，需要提前准备好所有容器镜像并导入到私有镜像仓库：

```bash
# 1. 在有网络的环境中拉取并保存镜像
docker pull infinilabs/operator:latest
docker pull infinilabs/easysearch:1.7.0-223
docker save infinilabs/operator:latest -o operator.tar
docker save infinilabs/easysearch:1.7.0-223 -o easysearch.tar

# 2. 将镜像文件传输到离线环境并导入
docker load -i operator.tar
docker load -i easysearch.tar

# 3. 推送到私有仓库
docker tag infinilabs/operator:latest registry.internal.com/infinilabs/operator:latest
docker push registry.internal.com/infinilabs/operator:latest
```

在 Operator Deployment 和 SearchCluster YAML 中使用私有仓库地址，并配置 `imagePullSecrets`：

```yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: my-registry-secret
      containers:
        - name: k8s-operator
          image: registry.internal.com/infinilabs/operator:latest
```

### 多集群管理

在同一 Kubernetes 集群中可以部署多个独立的 Easysearch 集群，通过不同的 `metadata.name` 和 namespace 来隔离：

```yaml
# 集群 A — 业务数据
apiVersion: infinilabs.infinilabs.com/v1
kind: SearchCluster
metadata:
  name: cluster-business
  namespace: easysearch-prod
# ...

# 集群 B — 日志分析
apiVersion: infinilabs.infinilabs.com/v1
kind: SearchCluster
metadata:
  name: cluster-logging
  namespace: easysearch-logs
# ...
```

> 每个 namespace 需要独立配置 RBAC、证书和密码 Secret。

### Operator 高可用

Operator 本身以 Deployment 方式部署，默认为单副本。Operator 是无状态的控制器，如果 Pod 异常退出，Kubernetes 会自动重新调度。

如需进一步增强可用性，可将 Operator 的 `replicas` 设为 2 并启用 Leader Election（需要 Operator 镜像支持）。在任一时刻只有一个实例作为 Leader 执行 Reconcile，另一个实例处于热备状态。

## 卸载与清理

### 删除 Easysearch 集群

```bash
# 删除 SearchCluster 资源（Operator 会自动清理关联的 StatefulSet、Service 等）
kubectl delete searchcluster threenodes -n default

# 注意：PVC 默认不会自动删除，需要手动清理
kubectl delete pvc data-threenodes-masters-0 data-threenodes-masters-1 data-threenodes-masters-2
```

> ⚠️ **删除 PVC 将永久丢失数据**，请确保已做好备份。

### 卸载 Operator

```bash
# 删除 Operator Deployment
kubectl delete deployment k8s-operator -n default

# 清理 RBAC 资源
kubectl delete clusterrolebinding manager-rolebinding
kubectl delete clusterrole manager-role
kubectl delete serviceaccount controller-manager -n default

# 清理证书资源（可选）
kubectl delete certificate ca-certificate easysearch-certs easysearch-admin-certs -n default
kubectl delete issuer selfsigned-issuer ca-issuer -n default
```

## 版本兼容

| Operator 版本 | Easysearch 版本 | Kubernetes 版本 | 说明 |
|---------------|-----------------|-----------------|------|
| latest | 1.6.x — 最新 | ≥ 1.9 | 查看 [GitHub Releases](https://github.com/infinilabs/operator/releases) 获取详细信息 |

> Operator 采用向前兼容策略，新版 Operator 可以管理旧版 Easysearch 集群。建议始终使用最新版本的 Operator。

## 子页面导航

| 页面 | 说明 |
|------|------|
| [部署 Operator]({{< relref "deploy_operator" >}}) | RBAC 配置、镜像下载、Deployment 部署 |
| [部署 Easysearch]({{< relref "deploy_easysearch" >}}) | SearchCluster YAML 编写、集群创建与验证 |
| [资源扩容]({{< relref "resource_manager" >}}) | CPU/内存/磁盘资源在线扩容（垂直扩容） |
| [节点扩容]({{< relref "node_scale" >}}) | 节点水平扩缩容操作 |
| [密码修改]({{< relref "update_password" >}}) | 通过 Secret 修改集群管理员密码 |
| [版本升级]({{< relref "upgrade" >}}) | 滚动升级 Easysearch 版本 |
| [证书管理]({{< relref "cert_manager" >}}) | cert-manager 证书配置与自动续期 |
| [S3 备份]({{< relref "s3_snapshot" >}}) | S3 定期快照备份配置 |
| [历史版本]({{< relref "history_version" >}}) | Operator 版本发布记录 |
| [FAQ]({{< relref "FAQ" >}}) | 常见问题与排查 |

## 背景知识

- [扩展 Kubernetes](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/)  
  Kubernetes 是高度可配置且可扩展的，在不修改它源码的情况下也可以新增业务自身的功能。
- [Operator 模式](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/operator/)  
  Operator 的核心是告诉 API 服务器，如何使现实与代码里配置的资源匹配。
- [定制资源 CRD](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/api-extension/custom-resources/)  
  除了 k8s 内置的资源比如 pod svc deployment 等，用户还可以自定义定制化的资源，并将定制资源与定制控制器相结合。
