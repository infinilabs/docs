---
title: "一键安装 Console 和 Easysearch"
date: 0001-01-01
summary: "一键安装 Console 和 Easysearch #  目前，Easysearch 已提供内置的 Console UI 来管理当前集群，如果您有多个集群需要管理，建议使用 INFINI Console 来统一管理多个 Easysearch 集群。INFINI Console 也可以通过一键安装脚本进行安装，以下是安装步骤：
# 启动默认配置 (Console + 1 个 Easysearch 节点) curl -fsSL http://get.infini.cloud/start-local | sh -s # 想要更丰富的体验？试试这个： # 启动 3 个 Easysearch 节点，设置密码，并开启 Agent 指标采集 curl -fsSL http://get.infini.cloud/start-local | sh -s -- up --nodes 3 --password &#34;MyDevPass123.&#34; --metrics-agent  具体参数说明请参考博客文章 一键启动：使用 start-local 脚本轻松管理 INFINI Console 与 Easysearch 本地环境。
 "
---


# 一键安装 Console 和 Easysearch

目前，Easysearch 已提供内置的 Console UI 来管理当前集群，如果您有多个集群需要管理，建议使用 INFINI Console 来统一管理多个 Easysearch 集群。INFINI Console 也可以通过一键安装脚本进行安装，以下是安装步骤：

```bash
# 启动默认配置 (Console + 1 个 Easysearch 节点)
curl -fsSL http://get.infini.cloud/start-local | sh -s

# 想要更丰富的体验？试试这个：
# 启动 3 个 Easysearch 节点，设置密码，并开启 Agent 指标采集
curl -fsSL http://get.infini.cloud/start-local | sh -s -- up --nodes 3 --password "MyDevPass123." --metrics-agent
```

> 具体参数说明请参考博客文章 [一键启动：使用 start-local 脚本轻松管理 INFINI Console 与 Easysearch 本地环境](https://www.infinilabs.cn/blog/2025/console-easysearch-start-local/)。
