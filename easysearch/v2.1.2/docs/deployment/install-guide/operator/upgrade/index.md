---
title: "版本升级"
date: 0001-01-01
summary: "版本升级 #  Easysearch Operator 支持通过修改 YAML 配置实现滚动升级，升级过程中集群保持可用。
操作步骤 #   查看当前版本  kubectl get pods -l app=easysearch -o jsonpath=&#39;{.items[0].spec.containers[0].image}&#39; 修改 Operator YAML 中的版本字段  # 修改前 version: &#34;1.7.0-223&#34; # 修改后 version: &#34;1.7.1-225&#34; 其他配置保持不变：
httpPort: 9200 vendor: Easysearch serviceAccount: controller-manager serviceName: threenodes 应用修改  kubectl apply -f easysearch-cluster.yaml 滚动升级流程 #  为保证升级过程中的服务可用性，Operator 采用滚动升级方式：
 从 threenodes-masters-0 开始升级 等待该节点完全就绪后，升级 threenodes-masters-1 依次滚动，直到所有节点升级完毕   整个升级过程耗时约 10 分钟（3 节点集群），具体取决于数据量和分片恢复速度。
 验证升级结果  # 进入容器验证版本 kubectl exec -it threenodes-masters-0 -- curl -ku admin:PASSWORD https://localhost:9200 确认 version."
---


# 版本升级

Easysearch Operator 支持通过修改 YAML 配置实现滚动升级，升级过程中集群保持可用。

## 操作步骤

1. **查看当前版本**

```bash
kubectl get pods -l app=easysearch -o jsonpath='{.items[0].spec.containers[0].image}'
```

2. **修改 Operator YAML 中的版本字段**

```yaml
# 修改前
version: "1.7.0-223"

# 修改后
version: "1.7.1-225"
```

其他配置保持不变：

```yaml
httpPort: 9200
vendor: Easysearch
serviceAccount: controller-manager
serviceName: threenodes
```

3. **应用修改**

```bash
kubectl apply -f easysearch-cluster.yaml
```

## 滚动升级流程

为保证升级过程中的服务可用性，Operator 采用**滚动升级**方式：

1. 从 `threenodes-masters-0` 开始升级
2. 等待该节点完全就绪后，升级 `threenodes-masters-1`
3. 依次滚动，直到所有节点升级完毕

> 整个升级过程耗时约 10 分钟（3 节点集群），具体取决于数据量和分片恢复速度。

4. **验证升级结果**

```bash
# 进入容器验证版本
kubectl exec -it threenodes-masters-0 -- curl -ku admin:PASSWORD https://localhost:9200
```

确认 `version.number` 已更新到目标版本。

## 操作演示

{{< asciinema key="/upgrade" autoplay="1" speed="2" rows="30" preload="1" >}}

