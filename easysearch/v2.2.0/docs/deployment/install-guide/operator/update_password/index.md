---
title: "密码修改"
date: 0001-01-01
summary: "密码修改 #  Easysearch Operator 将集群密码保存在 Kubernetes Secret 中。修改密码只需更新 Secret，Operator 会自动检测变更并应用。
操作步骤 #   查看现有 Secret  kubectl get secret threenodes-admin-password -o yaml 创建或修改密码 Secret 文件  apiVersion: v1 kind: Secret metadata: name: threenodes-admin-password type: Opaque data: # admin（base64 编码） username: YWRtaW4= # admin123（base64 编码） password: YWRtaW4xMjM=  提示：使用 echo -n &quot;your_password&quot; | base64 生成 base64 编码值。
 应用修改  kubectl apply -f admin-credentials-secret.yaml 自动生效流程 #  Operator 感知到 threenodes-admin-password Secret 变更后："
---


# 密码修改

Easysearch Operator 将集群密码保存在 Kubernetes Secret 中。修改密码只需更新 Secret，Operator 会自动检测变更并应用。

## 操作步骤

1. **查看现有 Secret**

```bash
kubectl get secret threenodes-admin-password -o yaml
```

2. **创建或修改密码 Secret 文件**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: threenodes-admin-password
type: Opaque
data:
  # admin（base64 编码）
  username: YWRtaW4=
  # admin123（base64 编码）
  password: YWRtaW4xMjM=
```

> **提示**：使用 `echo -n "your_password" | base64` 生成 base64 编码值。

3. **应用修改**

```bash
kubectl apply -f admin-credentials-secret.yaml
```

## 自动生效流程

Operator 感知到 `threenodes-admin-password` Secret 变更后：

1. 通过比较密码 hash 值与 Job annotations 中的 `securityconfig/checksum` 判断是否有更新
2. 如果检测到变更，自动重新执行 `threenodes-securityconfig-update` Job
3. 该 Job 通过管理员证书调用 Easysearch 安全 API 修改密码

```bash
# Job 内部执行的核心命令
curl -k -XPUT --cert admin-credentials/tls.crt --key admin-credentials/tls.key \
  -H 'Content-Type: application/json' \
  'https://threenodes.default.svc.cluster.local:9200/_security/user/admin' \
  -d '{"password": "admin123", "external_roles": ["admin"]}'
```

4. **验证新密码**

```bash
# 使用新密码测试
kubectl exec -it threenodes-masters-0 -- \
  curl -ku admin:admin123 https://localhost:9200/_cluster/health
```

## 操作演示

{{< asciinema key="/update_password" autoplay="1" speed="2" rows="30" preload="1" >}}

