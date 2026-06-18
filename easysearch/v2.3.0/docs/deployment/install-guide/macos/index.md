---
title: "macOS"
date: 0001-01-01
summary: "MacOS 环境下使用 Easysearch #  目前，有多种方案可以在 MacOS 下体验 Easysearch。可以选择使用 Docker 方式安装，或者使用本地安装包进行安装。
前置要求 #   macOS 10.15 (Catalina) 或更高版本 JDK 11+（推荐 JDK 17）。Bundle 包已内置 JDK，无需单独安装。 至少 4 GB 可用内存  方案一：Docker 安装（推荐） #  如果您的 MacOS 环境上有 Docker（Docker Desktop、OrbStack 等），可以用最简单的方式启动 Easysearch：
docker run -d --name easysearch \  -p 9200:9200 \  -e &#34;EASYSEARCH_INITIAL_ADMIN_PASSWORD=EasysearchP@ssw0rd!&#34; \  -e &#34;ES_JAVA_OPTS=-Xms512m -Xmx512m&#34; \  infinilabs/easysearch:latest  详细 Docker 配置请参考 Docker 环境下使用 Easysearch。
 方案二：本地安装 #  使用一键安装脚本进行安装，请按照以下步骤操作："
---


# MacOS 环境下使用 Easysearch

目前，有多种方案可以在 MacOS 下体验 Easysearch。可以选择使用 Docker 方式安装，或者使用本地安装包进行安装。

## 前置要求

- macOS 10.15 (Catalina) 或更高版本
- JDK 11+（推荐 JDK 17）。Bundle 包已内置 JDK，无需单独安装。
- 至少 4 GB 可用内存

## 方案一：Docker 安装（推荐）

如果您的 MacOS 环境上有 Docker（Docker Desktop、OrbStack 等），可以用最简单的方式启动 Easysearch：

```bash
docker run -d --name easysearch \
  -p 9200:9200 \
  -e "EASYSEARCH_INITIAL_ADMIN_PASSWORD=EasysearchP@ssw0rd!" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  infinilabs/easysearch:latest
```

> 详细 Docker 配置请参考 [Docker 环境下使用 Easysearch]({{< relref "./docker.md" >}})。

## 方案二：本地安装

使用一键安装脚本进行安装，请按照以下步骤操作：

```bash
# 打开终端，使用 bash 执行以下命令
mkdir -p /Users/$(whoami)/data/easysearch

# 下载最新版本的 Easysearch 并安装
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch -d /Users/$(whoami)/data/easysearch

# 进入 Easysearch 目录
cd /Users/$(whoami)/data/easysearch

# 初始化 Easysearch
bin/initialize.sh -s

# 运行 Easysearch
bin/easysearch -d -p pid
```

> **注意**：初始化过程中会生成随机密码，并不会保存到日志文件中，只会在终端显示一次，请妥善保存。

## 验证安装

```bash
# 使用初始化时输出的密码
curl -ku admin:YOUR_PASSWORD https://localhost:9200

# 预期输出包含
# "cluster_name" : "easysearch",
# "tagline" : "You Know, For Easy Search!"
```

## 常见问题

### 忘记 admin 密码

```bash
# 使用密码重置脚本
cd /Users/$(whoami)/data/easysearch
bin/reset_admin_password.sh
```

### macOS 安全提示

如果通过 Chrome 等浏览器下载并解压安装包，macOS 可能会给解压后的文件附加 `com.apple.quarantine` 隔离标签。此时启动时可能出现 `"java" cannot be opened because the developer cannot be verified` 或“无法验证开发者”的提示，即使 bundled JDK 的代码签名本身是合法的。

`bin/initialize.sh` 会在 macOS 上自动检测并清理 Easysearch 启动脚本和 bundled JDK 可执行文件上的隔离标签。正常情况下只需要重新执行初始化脚本：

```bash
bin/initialize.sh
```

如需手动确认某个文件是否带有隔离标签，可以执行：

```bash
xattr -p com.apple.quarantine jdk/Contents/Home/bin/java
```

如果初始化脚本提示自动清理失败，可手动清理相关可执行文件的隔离标签，例如：

```bash
xattr -d com.apple.quarantine bin/easysearch
xattr -d com.apple.quarantine jdk/Contents/Home/bin/java
```

### 停止 Easysearch

```bash
# 使用 PID 文件停止
kill $(cat /Users/$(whoami)/data/easysearch/pid)
```

## 延伸阅读

- [Docker 部署]({{< relref "./docker.md" >}})
- [测试环境部署]({{< relref "./test-env.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})

