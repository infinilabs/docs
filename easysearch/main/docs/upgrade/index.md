---
title: "升级手册"
date: 0001-01-01
description: "版本升级说明与迁移指南。"
summary: "升级手册 #  Easysearch 版本升级与迁移指南。
升级文档 #   从 1.15.x 迁移到 3.0.0：直接升级到 2.5.1，完成系统索引迁移与业务准备，再升级到 3.0.0 K8s Operator 升级：使用 Operator 管理的集群升级 版本历史：各版本变更与不兼容调整  升级前建议 #   查阅目标版本的 发布日志，了解变更与破坏性更新 在测试环境验证升级流程 做好数据备份  升级到 2.5.1：系统索引迁移 #  2.5.1 增加升级到 3.0.0 前的系统索引迁移能力。先将全部节点升级到 2.5.1，保留原数据、安全配置与节点身份， 再通过 GET /_migration/system_features 检查、POST /_migration/system_features 启动、GET 轮询结果。 1.15.x 集群可直接升级到 2.5.1，无需先安装 2.5.0；已经运行 2.5.0 的集群更新到 2.5.1 即可。 2.5.1 同时修复旧 ZSTD 索引混部写入误选只读格式的问题，支持保留旧段并在滚动升级期间继续读写。 无需额外迁移配置项；完整操作与备份要求见迁移指南。
沿用下文的 JDK 与安全 agent 配置要求，使用完整 2.5.1 包。2.5.1 起，发行包中的 agent 使用不带版本号的固定文件名； 保留旧 JVM 配置时，将原来带版本号的 agent 引用替换一次："
---


# 升级手册

Easysearch 版本升级与迁移指南。

## 升级文档

- **[从 1.15.x 迁移到 3.0.0]({{< relref "./migrate-1.15-to-3.0.md" >}})**：直接升级到 2.5.1，完成系统索引迁移与业务准备，再升级到 3.0.0
- **[K8s Operator 升级]({{< relref "../deployment/install-guide/operator/upgrade.md" >}})**：使用 Operator 管理的集群升级
- **[版本历史]({{< relref "../release-notes/" >}})**：各版本变更与不兼容调整

## 升级前建议

1. 查阅目标版本的 [发布日志]({{< relref "../release-notes/" >}})，了解变更与破坏性更新
2. 在测试环境验证升级流程
3. 做好数据备份

## 升级到 2.5.1：系统索引迁移

2.5.1 增加升级到 3.0.0 前的系统索引迁移能力。先将全部节点升级到 2.5.1，保留原数据、安全配置与节点身份，
再通过 `GET /_migration/system_features` 检查、`POST /_migration/system_features` 启动、GET 轮询结果。
1.15.x 集群可直接升级到 2.5.1，无需先安装 2.5.0；已经运行 2.5.0 的集群更新到 2.5.1 即可。
2.5.1 同时修复旧 ZSTD 索引混部写入误选只读格式的问题，支持保留旧段并在滚动升级期间继续读写。
无需额外迁移配置项；完整操作与备份要求见[迁移指南]({{< relref "./migrate-1.15-to-3.0.md#三在-251-中迁移系统索引" >}})。

沿用下文的 JDK 与安全 agent 配置要求，使用完整 2.5.1 包。2.5.1 起，发行包中的 agent 使用不带版本号的固定文件名；
保留旧 JVM 配置时，将原来带版本号的 agent 引用替换一次：

```text
21-:-javaagent:lib/easysearch-security-agent.jar
```

后续升级无需再因 agent 文件名中的版本号修改该行；仍须使用新发行包配套的 agent 和 bootstrap JAR，并同步其它必要 JVM 参数。

只有迁移任务创建后才启用持久版本门槛，此后低于 2.5.1 的节点不能再加入，迁移结束也不会取消该限制。
不准备升级 3.0.0 的集群，无需仅因更新到 2.5.1 就立即执行迁移；符合原版本兼容条件的旧业务索引仍按 2.5 系列行为读写。

## 升级到 2.5.0：JDK 与 JVM 安全配置

Easysearch 2.5.0 默认使用并推荐 **JDK 25**，Bundle 包默认内置 JDK 25。
对于已部署 JDK 21 且暂不方便升级 JDK 的客户环境，保留 JDK 21 兼容运行能力。
安全 agent 在 JDK 21 及以上统一加载，并要求禁用 JDK 自身的 SecurityManager。
JDK 26、27 已通过发行包实际启动验证，可正常运行；切换 JDK 时仍需同步下述必要启动参数。
详见[安装指南]({{< relref "../deployment/install-guide/_index.md" >}})。

如果升级时保留了旧的 `config/jvm.options`、`config/jvm.options.d/*.options` 或自定义 `ES_PATH_CONF`，
需对照新发行包逐项同步必要参数，不能只替换程序文件：

1. 保留 `-Djava.security.manager=disallow`，移除旧的 `allow`、启用 SecurityManager 的参数及其他冲突覆盖。
   也检查 `ES_JAVA_OPTS` 等额外参数。启动时会检查该属性严格为 `disallow` 且没有安装 JDK SecurityManager，否则拒绝启动。
2. 将旧的 `21:`、`25:` 两条 agent 配置替换为新包的单条 `21-:` 配置，并同步 jar 文件名：

   ```text
   21-:-javaagent:lib/easysearch-security-agent-2.5.0.jar
   ```

   上例用于 2.5.0；2.5.1 起使用上一节的固定文件名 `easysearch-security-agent.jar`。
   `21-:` 在 JDK 21 及以上生效。移除旧的重复引用，并检查 `jvm.options.d/` 和 `ES_JAVA_OPTS`，
   确保每次启动只加载一个安全 agent。保留随包提供的 bootstrap jar，避免混用旧版本文件。
   Windows NIO Selector 临时 socket 清理权限修复还涉及核心程序和相关模块策略，需使用配套的完整发行包，
   不能仅替换 agent jar 或修改 `jvm.options`。
3. 保留新包的 `21-:--enable-preview`、`19-:--enable-native-access=ALL-UNNAMED` 等必要 JVM 参数，
   再迁移自定义堆大小、GC 日志路径等配置。不要直接用旧配置覆盖新包配置。
   JVMCI 编译器相关的解锁、禁用及重新锁定三行使用 `21-26:` 条件；JDK 27 不再识别 `UseJVMCICompiler`，
   旧的 `21-:-XX:-UseJVMCICompiler` 会导致 JVM 启动失败。
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
