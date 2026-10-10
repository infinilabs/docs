---
title: "升级手册"
date: 0001-01-01
description: "版本升级说明与迁移指南。"
summary: "升级手册 #  Easysearch 版本升级与迁移指南。
升级文档 #   K8s Operator 升级：使用 Operator 管理的集群升级 版本历史：各版本变更与不兼容调整  升级前建议 #   查阅目标版本的 发布日志，了解变更与破坏性更新 在测试环境验证升级流程 做好数据备份  升级到 2.5.0：JDK 与 JVM 安全配置 #  Easysearch 2.5.0 默认使用并推荐 JDK 25，Bundle 包默认内置 JDK 25。 对于已部署 JDK 21 且暂不方便升级 JDK 的客户环境，保留 JDK 21 兼容运行能力。 两种运行时都使用发行包提供的安全 agent，并要求禁用 JDK 自身的 SecurityManager。
如果升级时保留了旧的 config/jvm.options、config/jvm.options.d/*.options 或自定义 ES_PATH_CONF， 需对照新发行包逐项同步必要参数，不能只替换程序文件：
  保留 -Djava.security.manager=disallow，移除旧的 allow、启用 SecurityManager 的参数及其他冲突覆盖。 也检查 ES_JAVA_OPTS 等额外参数。启动时会检查该属性严格为 disallow 且没有安装 JDK SecurityManager，否则拒绝启动。"
---


# 升级手册

Easysearch 版本升级与迁移指南。

## 升级文档

- **[K8s Operator 升级]({{< relref "../deployment/install-guide/operator/upgrade.md" >}})**：使用 Operator 管理的集群升级
- **[版本历史]({{< relref "../release-notes/" >}})**：各版本变更与不兼容调整

## 升级前建议

1. 查阅目标版本的 [发布日志]({{< relref "../release-notes/" >}})，了解变更与破坏性更新
2. 在测试环境验证升级流程
3. 做好数据备份

## 升级到 2.5.0：JDK 与 JVM 安全配置

Easysearch 2.5.0 默认使用并推荐 **JDK 25**，Bundle 包默认内置 JDK 25。
对于已部署 JDK 21 且暂不方便升级 JDK 的客户环境，保留 JDK 21 兼容运行能力。
两种运行时都使用发行包提供的安全 agent，并要求禁用 JDK 自身的 SecurityManager。

如果升级时保留了旧的 `config/jvm.options`、`config/jvm.options.d/*.options` 或自定义 `ES_PATH_CONF`，
需对照新发行包逐项同步必要参数，不能只替换程序文件：

1. 保留 `-Djava.security.manager=disallow`，移除旧的 `allow`、启用 SecurityManager 的参数及其他冲突覆盖。
   也检查 `ES_JAVA_OPTS` 等额外参数。启动时会检查该属性严格为 `disallow` 且没有安装 JDK SecurityManager，否则拒绝启动。
2. 将 JDK 21、JDK 25 的 agent 引用同步为新包中的文件名：

   ```text
   21:-javaagent:lib/easysearch-security-agent-2.5.0.jar
   25:-javaagent:lib/easysearch-security-agent-2.5.0.jar
   ```

   两行分别仅在对应 JDK 上生效，每次启动加载一个安全 agent。保留随包提供的 bootstrap jar，避免混用旧版本文件。
3. 保留新包的 `21-:--enable-preview`、`19-:--enable-native-access=ALL-UNNAMED` 等必要 JVM 参数，
   再迁移自定义堆大小、GC 日志路径等配置。不要直接用旧配置覆盖新包配置。
4. 确认实际选中的 JDK。Linux/macOS 启动器使用 `jdk/` > `ES_JAVA_HOME` > `JAVA_HOME`；
   Windows `bin\easysearch.bat` 使用 `jdk/` > `JAVA_HOME`。已有 `jdk/` 时，仅设置环境变量不会切换运行时。

启动后可核对各节点实际运行时：

```bash
# 使用现有管理员密码
curl -ku 'admin:YOUR_PASSWORD' 'https://localhost:9200/_cat/nodes?h=name,jdk&v'
```

升级时保留原有数据目录、认证与 TLS 配置；上述 JVM 安全配置不要求重新生成证书或管理员密码。
`security.enabled: false` 也不会关闭安全 agent 或取消 SecurityManager 禁用校验。
详细参数与安全检查范围见[JVM 配置]({{< relref "../deployment/config/node-settings/jvm.md" >}})。
