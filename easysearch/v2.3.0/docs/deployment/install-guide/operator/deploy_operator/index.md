---
title: "部署 Operator"
date: 0001-01-01
summary: "部署 Easysearch Operator #  Easysearch Operator 只能在 k8s 环境下部署安装，请准备好一套 k8s 环境
部署前准备 #   k8s 环境
要求Kubernetes 1.9以上版本，自 1.9 版本以后，StatefulSet成为了在Kubernetes中管理有状态应用的标准方式。 StorageClass
StorageClass 允许集群管理员定义多种存储方案，如快速的 SSD、标准的硬盘，或者其他的存储系统。无需手动预先创建存储资源，用户只需要在 PersistentVolumeClaim (PVC) 中指定需要的 StorageClass，存储资源就可以根据需求动态地创建。 ServiceAccount
创建一个 ServiceAccount 用于 Easysearch Operator 获取和操作 k8s 资源 apiVersion: v1 kind: ServiceAccount metadata: labels: app.kubernetes.io/name: serviceaccount app.kubernetes.io/instance: controller-manager-sa app.kubernetes.io/component: rbac app.kubernetes.io/created-by: k8s-operator app.kubernetes.io/part-of: k8s-operator app.kubernetes.io/managed-by: kustomize name: controller-manager # ServiceAccount 的名字是 controller-manager namespace: default  ClusterRole
创建 ClusterRole，用于定义访问 k8s 集群的角色权限 展开查看完整代码 apiVersion: rbac."
---


# 部署 **Easysearch Operator**

**Easysearch Operator** 只能在 k8s 环境下部署安装，请准备好一套 k8s 环境

## 部署前准备

- k8s 环境  
  要求`Kubernetes 1.9`以上版本，自 1.9 版本以后，`StatefulSet`成为了在`Kubernetes`中管理有状态应用的标准方式。
- StorageClass  
  StorageClass 允许集群管理员定义多种存储方案，如快速的 SSD、标准的硬盘，或者其他的存储系统。无需手动预先创建存储资源，用户只需要在 PersistentVolumeClaim (PVC) 中指定需要的 StorageClass，存储资源就可以根据需求动态地创建。
- ServiceAccount  
  创建一个 ServiceAccount 用于 Easysearch Operator 获取和操作 k8s 资源
  ```yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
  labels:
  app.kubernetes.io/name: serviceaccount
  app.kubernetes.io/instance: controller-manager-sa
  app.kubernetes.io/component: rbac
  app.kubernetes.io/created-by: k8s-operator
  app.kubernetes.io/part-of: k8s-operator
  app.kubernetes.io/managed-by: kustomize
  name: controller-manager # ServiceAccount 的名字是 controller-manager
  namespace: default
  ```
- ClusterRole  
  创建 ClusterRole，用于定义访问 k8s 集群的角色权限
  {{< details "展开查看完整代码" "..." >}}
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
  name: manager-role
  namespace: default
  rules:
  - apiGroups:
    - apps
      resources:
    - deployments
      verbs:
    - create
    - delete
    - get
    - list
    - patch
    - update
    - watch
    - apiGroups:
      - apps
        resources:
      - statefulsets
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - batch
        resources:
      - jobs
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - ""
        resources:
      - configmaps
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - ""
        resources:
      - events
        verbs:
      - create
      - patch
      - update
    - apiGroups:
      - ""
        resources:
      - namespaces
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - ""
        resources:
      - pods
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - ""
        resources:
      - secrets
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - ""
        resources:
      - services
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - infinilabs.infinilabs.com
        resources:
      - events
        verbs:
      - create
      - patch
    - apiGroups:
      - infinilabs.infinilabs.com
        resources:
      - searchclusters
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - infinilabs.infinilabs.com
        resources:
      - searchclusters/finalizers
        verbs:
      - update
    - apiGroups:
      - infinilabs.infinilabs.com
        resources:
      - searchclusters/status
        verbs:
      - get
      - patch
      - update
    - apiGroups:
      - policy
        resources:
      - poddisruptionbudgets
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - monitoring.coreos.com
        resources:
      - servicemonitors
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
    - apiGroups:
      - ""
        resources:
      - persistentvolumeclaims
        verbs:
      - create
      - delete
      - get
      - list
      - patch
      - update
      - watch
  ```
- ClusterRoleBinding  
  创建 ClusterRoleBinding，将 service account（controller-manager） 和 role （manager-role）绑定起来
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    labels:
      app.kubernetes.io/name: clusterrolebinding
      app.kubernetes.io/instance: manager-rolebinding
      app.kubernetes.io/component: rbac
      app.kubernetes.io/created-by: k8s-operator
      app.kubernetes.io/part-of: k8s-operator
      app.kubernetes.io/managed-by: kustomize
      name: manager-rolebinding
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: manager-role
  subjects:
    - kind: ServiceAccount
      name: controller-manager
      namespace: default
  ```
  {{< /details >}}
- cert-manager  
  部署 cert-manager 来管理 Easysearch 整个集群的证书

至此，准备工作完毕，以上的流程都是一劳永逸的，在第一次部署 Operator 的时候才需要，后续不再需要重新部署。

## 部署 Easysearch Operator

### 下载镜像

**Easysearch Operator** 的镜像发布在 Docker 的官方仓库，地址如下：

https://hub.docker.com/r/infinilabs/operator

使用下面的命令即可获取最新的容器镜像：

```bash
docker pull infinilabs/operator:latest
```

### 验证镜像

将镜像下载到本地之后，可以看到 **Easysearch Operator** 容器镜像非常小，只有大概 60MB，所以下载的速度应该是非常快的。

```text
➜ docker images
REPOSITORY             TAG      IMAGE ID       CREATED         SIZE
infinilabs/operator   latest   5da94b93e7eb   8 minutes ago   61.2MB
```

### 部署 Operator

在已有的 k8s 环境中部署 Operator，Operator 采用 deployment 方式部署，编辑对应的 yaml 文件，并 apply 即可。
```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: k8s-operator
  namespace: default
  labels:
    app: k8s-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: k8s-operator
  template:
    metadata:
      labels:
        app: k8s-operator
    spec:
      containers:
        - name: k8s-operator
          image: 'infinilabs/operator:latest'
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      serviceAccount: controller-manager # 指定账号
```

### 查看部署情况

```shell
ik get deployment

NAME           READY   UP-TO-DATE   AVAILABLE   AGE
k8s-operator   1/1     1            1           2m6s
```

至此，Easysearch Operator 已经部署完成。

