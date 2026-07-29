---
title: "S3 备份"
date: 0001-01-01
summary: "S3 定期备份 #  Easysearch Operator 支持通过 YAML 配置来管理 S3 快照备份策略。Operator 会启动一个 Job，通过集群 API 自动配置相应的备份策略。
工作原理 #   在 Operator YAML 中配置 S3 备份相关参数（Bucket 名称、访问密钥、备份周期等） Operator 创建一个 Job，该 Job 调用 Easysearch 的快照生命周期管理（SLM）API SLM 按照配置的策略自动执行定期快照  相关文档 #    快照生命周期管理 API  备份与恢复  "
---


# S3 定期备份

Easysearch Operator 支持通过 YAML 配置来管理 S3 快照备份策略。Operator 会启动一个 Job，通过集群 API 自动配置相应的备份策略。

## 工作原理

1. 在 Operator YAML 中配置 S3 备份相关参数（Bucket 名称、访问密钥、备份周期等）
2. Operator 创建一个 Job，该 Job 调用 Easysearch 的快照生命周期管理（SLM）API
3. SLM 按照配置的策略自动执行定期快照

## 相关文档

- [快照生命周期管理 API]({{< relref "/docs/features/data-retention/slm" >}})
- [备份与恢复]({{< relref "/docs/features/data-retention/backup-restore" >}})

