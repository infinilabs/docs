---
title: "部署 Easysearch"
date: 0001-01-01
summary: "部署Easysearch Operator #  这里我们准备部署一个 3 节点的Easysearch 集群，准备 three-nodes-easysearch-cluster.yaml 文件，文件内容如下所示，并对关键字段都进行了注释。
apiVersion: infinilabs.infinilabs.com/v1 kind: SearchCluster # 自定义的资源类型 metadata: name: threenodes # Easysearch 集群的名称 namespace: default # Easysearch 集群所在的命名空间 spec: # 规格 security: # 安全相关 config: adminSecret: # admin证书配置 name: easysearch-admin-certs adminCredentialsSecret: # 账户密码配置 name: threenodes-admin-password tls: # tls 协议配置，包括节点间的transport，以及访问集群的http http: # 访问集群http配置 generate: false # 是否需要集群自动生成证书 secret: # 自定义证书配置 name: easysearch-certs transport: # 集群间访问配置 generate: false perNode: false # 是否给每一个节点配置证书 secret: # 自定义证书配置 name: easysearch-certs nodesDn: [&#34;CN=Easysearch_Node&#34;] adminDn: [&#34;CN=Easysearch_Admin&#34;] general: # 通用配置 snapshotRepositories: # s3 快照配置 - name: s3_repository # 配置的s3快照的名称 type: s3 # 快照类型 settings: # 快照配置 bucket: es-operator-bucket # s3中的桶，需提前建好 access_key: minioadmin # 访问s3密钥 secret_key: minioadmin endpoint: http://192."
---


# 部署**Easysearch Operator**

这里我们准备部署一个 3 节点的`Easysearch` 集群，准备 `three-nodes-easysearch-cluster.yaml` 文件，文件内容如下所示，并对关键字段都进行了注释。

```yaml
apiVersion: infinilabs.infinilabs.com/v1
kind: SearchCluster # 自定义的资源类型
metadata:
  name: threenodes # Easysearch 集群的名称
  namespace: default # Easysearch 集群所在的命名空间
spec: # 规格
  security: # 安全相关
    config:
      adminSecret: # admin证书配置
        name: easysearch-admin-certs
      adminCredentialsSecret: # 账户密码配置
        name: threenodes-admin-password
    tls: # tls 协议配置，包括节点间的transport，以及访问集群的http
      http: # 访问集群http配置
        generate: false # 是否需要集群自动生成证书
        secret: # 自定义证书配置
          name: easysearch-certs
      transport: # 集群间访问配置
        generate: false
        perNode: false # 是否给每一个节点配置证书
        secret: # 自定义证书配置
          name: easysearch-certs
        nodesDn: ["CN=Easysearch_Node"]
        adminDn: ["CN=Easysearch_Admin"]
  general: # 通用配置
    snapshotRepositories: # s3 快照配置
      - name: s3_repository # 配置的s3快照的名称
        type: s3 # 快照类型
        settings: # 快照配置
          bucket: es-operator-bucket # s3中的桶，需提前建好
          access_key: minioadmin # 访问s3密钥
          secret_key: minioadmin
          endpoint: http://192.168.3.185:19000 # s3 访问地址
          # compress: true
    version: "1.7.0-223" # Easysearch 版本
    httpPort: 9200 # Easysearch 监听http端口
    vendor: Easysearch
    serviceAccount: controller-manager # 访问k8s的service account，需要提前配置好，主要用于访问k8s资源
    serviceName: threenodes
    monitoring:
      enable: false
    pluginsList: []
    drainDataNodes: false # 在删除节点之前，是否需要先将节点上的数据迁移出去
  nodePools: # Easysearch 节点池
    - component: masters # 组件名称，作为标识，比如可以是masters，snapshotconfig securityconfig
      replicas: 3 # 配置集群节点个数
      diskSize: "30Gi" # 每个节点占多少磁盘空间
      jvm: -Xmx4G -Xms4G # jvm 参数
      nodeSelector: # 调度时选择node
      resources: # 节点所需资源配置，包括申请值和最大值
        requests:
          memory: "4Gi"
          cpu: "1"
        limits:
          memory: "5Gi"
          cpu: "2"
      roles: # 节点的角色，一般有master主节点，data数据节点，以及两者可
        - "master"
        - "data"
```

编写好上述的 Operator yaml 文件，执行命令：

```shell
kubectl create -f  three-nodes-easysearch-cluster.yaml
```

为了快速组建集群，首先会创建一个 bootstrap 节点，然后各个节点同时并发启动，等待一段时间后集群创建成功，分别创建 3 个节点：

```text
threenodes-masters-0
threenodes-masters-1
threenodes-masters-2
```

查看 pvc，对应到 3 个节点分别创建了 3 个 pvc:

```text
data-threenodes-masters-0
data-threenodes-masters-1
data-threenodes-masters-2
```

如下图：

查看 secret，已经创建了 `threenodes-admin-password`，用于保存账号密码

进入 console-0，查看集群状态

```shell
curl -ku admin:xxxxxxxxxxxx https://threenodes.default.svc.cluster.local:9200

{
  "name" : "threenodes-masters-0",
  "cluster_name" : "threenodes",
  "cluster_uuid" : "WIuPcQ2gR4aG2hFrTLF4NQ",
  "version" : {
    "distribution" : "easysearch",
    "number" : "1.7.0",
    "distributor" : "INFINI Labs",
    "build_hash" : "b8a4c52487d245a65679fa4dc45db166cb919c4d",
    "build_date" : "2023-12-15T01:52:39.584409Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.2",
    "minimum_wire_lucene_version" : "7.7.0",
    "minimum_lucene_index_compatibility_version" : "7.7.0"
  },
  "tagline" : "You Know, For Easy Search!"
}
```

可继续通过 console web 端操作集群

至此，说明整个集群成功运行起来。

{{< asciinema key="/deploy_operator" autoplay="1" speed="2" rows="30" preload="1" >}}

https://asciinema.org/a/cvWISVzr23CPiHYeZwexF4DnP

