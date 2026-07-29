---
title: "Helm Chart"
date: 0001-01-01
summary: "Helm Chart 部署 #  INFINI Easysearch 从 1.5.0 版本开始支持 Helm Chart 方式部署。
仓库信息 #  INFINI Easysearch Helm Chart 仓库地址: https://helm.infinilabs.com。
可以使用以下命令添加仓库
helm repo add infinilabs https://helm.infinilabs.com 依赖项 #   StorageClass  INFINI Easysearch Helm Chart 包中默认使用 local-path 进行数据持久化存储，可参考 local-path官方文档进行安装。
如果使用其他 StorageClass，请修改 Chart 包中的 storageClassName: local-path配置项。
 Secret  INFINI Easysearch Helm Chart 默认使用 cert-manager 进行自签 CA 证书创建及分发, 可参考 cert-manager 官方文档进行安装。
安装示例 #  cat &lt;&lt; EOF | kubectl apply -n &lt;namespace&gt; -f - apiVersion: cert-manager."
---


# Helm Chart 部署

INFINI Easysearch 从 1.5.0 版本开始支持 Helm Chart 方式部署。

## 仓库信息

INFINI Easysearch Helm Chart 仓库地址: [https://helm.infinilabs.com](https://helm.infinilabs.com/)。

可以使用以下命令添加仓库

```bash
helm repo add infinilabs https://helm.infinilabs.com
```

## 依赖项

- StorageClass

INFINI Easysearch Helm Chart 包中默认使用 local-path 进行数据持久化存储，可参考[local-path](https://github.com/rancher/local-path-provisioner)官方文档进行安装。

如果使用其他 StorageClass，请修改 Chart 包中的 `storageClassName: local-path`配置项。

- Secret

INFINI Easysearch Helm Chart 默认使用 cert-manager 进行自签 CA 证书创建及分发, 可参考[cert-manager](https://cert-manager.io/docs/installation/) 官方文档进行安装。

## 安装示例

```bash
cat << EOF | kubectl apply -n <namespace> -f -
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: easysearch-ca-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: easysearch-ca-certificate
spec:
  commonName: easysearch-ca-certificate
  duration: 87600h0m0s
  isCA: true
  issuerRef:
    kind: Issuer
    name: easysearch-ca-issuer
  privateKey:
    algorithm: ECDSA
    size: 256
  renewBefore: 2160h0m0s
  secretName: easysearch-ca-secret
EOF

#安装 INFINI Easysearch, 默认单节点
helm install easysearch infinilabs/easysearch -n <namespace> --create-namespace
```

## 卸载

```bash
helm uninstall easysearch -n <namespace>
kubectl delete pvc easysearch-data-easysearch-0 easysearch-config-easysearch-0 -n <namespace>
```

> 更多信息请参考技术博客，[通过 Helm Chart 部署 Easysearch](https://www.infinilabs.com/blog/2023/use-helm-install-easysearch/)

