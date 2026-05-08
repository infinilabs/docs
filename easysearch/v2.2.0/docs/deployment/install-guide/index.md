---
title: "安装指南"
date: 0001-01-01
summary: "安装指南 #  本章介绍如何安装和运行 Easysearch。无论是开发测试还是生产部署，都可以在这里找到适合的安装方式。
 快速开始 #  最快的方式是使用一键安装脚本，在 Linux 上几分钟内完成安装：
# 1. 安装 Easysearch（默认安装到 /data/easysearch） curl -sSL http://get.infini.cloud | bash -s -- -p easysearch # 2. 初始化（生成证书和 admin 密码） cd /data/easysearch &amp;&amp; bin/initialize.sh -s # 3. 调整目录权限 chown -R easysearch:easysearch /data/easysearch # 4. 以 easysearch 用户启动 su easysearch -c &#34;/data/easysearch/bin/easysearch -d&#34; # 5. 验证（使用初始化时终端输出的密码） curl -ku admin:YOUR_PASSWORD https://localhost:9200  安全提示：初始化完成后 admin 密码将直接输出在终端中，只显示一次，请务必妥善保存。也可以通过环境变量 EASYSEARCH_INITIAL_ADMIN_PASSWORD 预先指定自定义密码（需要 1.8.2 及以后版本）。
 展开查看完整安装演示代码 # 使用 root 用户操作 whoami &amp;&amp; cat /etc/redhat-release &amp;&amp; uptime # 安装 JDK yum -y install java-11 # 创建 easysearch 用户 groupadd -g 602 easysearch useradd -u 602 -g easysearch -m -d /home/easysearch -c &#39;easysearch&#39; -s /bin/bash easysearch # 安装 Easysearch curl -sSL http://get."
---


# 安装指南

本章介绍如何安装和运行 Easysearch。无论是开发测试还是生产部署，都可以在这里找到适合的安装方式。

---

## 快速开始

最快的方式是使用一键安装脚本，在 Linux 上几分钟内完成安装：

```bash
# 1. 安装 Easysearch（默认安装到 /data/easysearch）
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch

# 2. 初始化（生成证书和 admin 密码）
cd /data/easysearch && bin/initialize.sh -s

# 3. 调整目录权限
chown -R easysearch:easysearch /data/easysearch

# 4. 以 easysearch 用户启动
su easysearch -c "/data/easysearch/bin/easysearch -d"

# 5. 验证（使用初始化时终端输出的密码）
curl -ku admin:YOUR_PASSWORD https://localhost:9200
```

> **安全提示**：初始化完成后 admin 密码将直接输出在终端中，**只显示一次**，请务必妥善保存。也可以通过环境变量 `EASYSEARCH_INITIAL_ADMIN_PASSWORD` 预先指定自定义密码（需要 1.8.2 及以后版本）。

{{< details "展开查看完整安装演示代码" "..." >}}

```bash
# 使用 root 用户操作
whoami && cat /etc/redhat-release && uptime
# 安装 JDK
yum -y install java-11
# 创建 easysearch 用户
groupadd -g 602 easysearch
useradd -u 602 -g easysearch -m -d /home/easysearch -c 'easysearch' -s /bin/bash easysearch
# 安装 Easysearch
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch
# 配置 Easysearch JDK
ln -s /usr/lib/jvm/java-11-openjdk-11.0.20.0.8-1.el7_9.x86_64 /data/easysearch/jdk
sed -i 's/1g/512m/g' /data/easysearch/config/jvm.options
# 初始化
cd /data/easysearch && bin/initialize.sh -s
# 调整目录权限
chown -R easysearch:easysearch /data/easysearch
# 运行 Easysearch
su easysearch -c "/data/easysearch/bin/easysearch -d"
# 检查 Easysearch，密码是随机的在安装过程中有展示
curl -ku admin:8bf1565cf6ff084d823e https://localhost:9200
```

{{< /details >}}

{{< asciinema key="/install" autoplay="1" speed="2" rows="30" preload="1" >}}

---

## 前置要求

### 操作系统

Easysearch 支持 Linux（x86_64 / ARM64 / LoongArch）、macOS 和 Windows。生产环境推荐 Linux。

### JDK 要求

| Easysearch 版本 | JDK 最低要求 | 推荐 JDK | 说明 |
|:----------------|:-----------:|:--------:|------|
| **2.0.3 及以上** | JDK 21 | JDK 21+ | 新版本强制要求 |
| **1.x ~ 2.0.2** | JDK 11 | JDK 17 | JDK 15+ 性能更优 |

> **推荐使用 Bundle 包**（内置 JDK），免去手动配置 JDK 的步骤。Bundle 包[下载地址](https://release.infinilabs.com/easysearch/stable/bundle/)。
>
> 如需使用自行安装的 JDK，有两种方式：
> 1. 将 `JAVA_HOME` 环境变量指向 JDK 安装路径
> 2. 将 JDK 软链接到 Easysearch 安装目录下并命名为 `jdk`

### JVM 堆内存

通过启动参数或配置文件设置 JVM 堆内存：

```bash
# 方式一：启动参数
ES_JAVA_OPTS="-Xms4g -Xmx4g" bin/easysearch

# 方式二：编辑 config/jvm.options
-Xms4g
-Xmx4g
```

> `-Xms` 和 `-Xmx` 建议设置为**相同值**，不超过物理内存的 50%，且**不超过 31 GB**。

### 系统调优

安装前请完成操作系统参数调优，否则 Easysearch 可能启动失败或性能不佳：

```bash
# 虚拟内存映射（必须）
sysctl -w vm.max_map_count=262144

# 文件描述符
ulimit -n 65535
```

> 完整调优参数请参考 [系统调优]({{< relref "/docs/deployment/config/settings" >}})。

---

## 选择安装方式

根据您的使用场景选择合适的安装方式：

### 按场景选择

| 场景 | 推荐方式 | 说明 |
|------|----------|------|
| 快速体验 / 开发测试 | [测试环境部署](./test-env.md) | 单节点，最快上手 |
| Docker 快速体验 | [Docker 部署](./docker.md) | 一条命令启动 |
| 本地多节点集群 | [Docker Compose](./docker-compose.md) | 2~3 节点模拟集群 |
| 生产环境 | [生产环境部署](./production-env.md) | 完整的上线检查清单 |
| 分布式集群 | [分布式集群部署](./cluster.md) | 自动化脚本搭建 |
| 内网 / 离线环境 | [离线安装](./offline-install.md) | 无需联网 |

### 按平台选择

**操作系统**

| 平台 | 链接 | 说明 |
|------|------|------|
| Linux | [Linux 安装](./linux.md) | 手动安装、Bundle 包、systemd 服务 |
| macOS | [macOS 安装](./macos.md) | 本地安装或 Docker |
| Windows | [Windows 安装](./windows.md) | 支持 Docker、手工安装、Git Bash |

**容器与编排**

| 方式 | 链接 | 说明 |
|------|------|------|
| Docker | [Docker 部署](./docker.md) | 单容器运行 |
| Docker Compose | [Docker Compose 集群](./docker-compose.md) | 多节点容器集群 |
| Helm Chart | [Helm Chart 部署](./helm.md) | Kubernetes 环境 |
| Operator | [Easysearch Operator](./operator/) | K8s 自动化运维 |

**云平台**

| 云厂商 | 链接 | 说明 |
|--------|------|------|
| 阿里云 | [阿里云部署](./aliyun.md) | ECS + ESSD 推荐配置 |
| AWS | [AWS 部署](./aws.md) | EC2 + gp3 推荐配置 |
| 腾讯云 | [腾讯云部署](./tencent-cloud.md) | CVM + 增强型 SSD 推荐配置 |

**信创平台**

Easysearch 全面支持国产芯片与操作系统：

| 芯片 / 平台 | 链接 |
|-------------|------|
| 龙芯（LoongArch） | [龙芯平台](ciip/loongson.md) |
| 鲲鹏（ARM） | [鲲鹏平台](ciip/kunpeng.md) |
| 飞腾（ARM） | [飞腾平台](ciip/feiteng.md) |
| 海光（x86） | [海光平台](ciip/hygon.md) |
| 兆芯（x86） | [兆芯平台](ciip/zhaoxin.md) |
| 申威 | [申威平台](ciip/shenwei.md) |
| 麒麟软件 | [麒麟软件平台](ciip/kylin.md) |
| 统信 UOS | [统信 UOS 平台](ciip/uniontech.md) |
| 移动云 BC-Linux | [BC-Linux 平台](ciip/bclinux.md) |

> 更多信创适配信息请参考博客：[Arm 架构](https://www.infinilabs.cn/blog/2023/easysearch-runing-with-arm/)、[LoongArch 架构](https://www.infinilabs.cn/blog/2023/easysearch-runing-with-loongarch/)。

**其他**

- [一键安装 Console 和 Easysearch](./console.md) — 同时部署 INFINI Console 管理界面
- [IPv6 支持](./ipv6.md) — 配置 IPv6 网络环境

---

## 初始化与安全

Easysearch **默认启用安全功能**，首次安装后需执行初始化：

```bash
cd /data/easysearch
bin/initialize.sh -s
```

初始化脚本会：
1. 生成 TLS 证书（节点间通信和 HTTPS）
2. 生成 admin 用户的随机密码
3. 安装默认插件

> ⚠️ **重要**：
> - admin 密码**仅在终端输出中显示一次**，请务必记录
> - 初始化脚本会覆盖已有的证书和密码，请勿对已运行的集群重复执行
> - 如忘记密码，可使用 `bin/reset_admin_password.sh` 重置，或参考[密码重置指南](https://infinilabs.cn/blog/2025/easysearch-reset-admin-password/)

---

## 验证安装

```bash
# 访问 REST API（将 YOUR_PASSWORD 替换为初始化时输出的密码）
curl -ku admin:YOUR_PASSWORD https://localhost:9200
```

预期输出：

```json
{
  "name" : "node-1",
  "cluster_name" : "easysearch",
  "cluster_uuid" : "...",
  "version" : {
    "distribution" : "easysearch",
    "number" : "x.y.z",
    "distributor" : "INFINI Labs",
    ...
  },
  "tagline" : "You Know, For Easy Search!"
}
```

> 也可以在浏览器中打开 `https://localhost:9200/` 验证（需要接受自签名证书提示）。

推荐使用 [INFINI Console](https://infinilabs.cn/docs/latest/console/) 进行集群管理，功能更加强大和方便。

---

## 插件安装

联网环境下，初始化脚本会自动安装常用插件。内网环境需要手动安装。

最新版本插件[下载地址](https://release.infinilabs.com/easysearch/stable/plugins/)。

```bash
# 切换到 easysearch 用户
su - easysearch

# 离线安装插件（以 analysis-ik 为例）
bin/easysearch-plugin install file:///tmp/analysis-ik-1.2.0.zip

# 查看已安装的插件
bin/easysearch-plugin list
```

常用插件列表：

| 插件 | 说明 |
|------|------|
| `analysis-ik` | 中文分词（IK Analyzer） |
| `analysis-pinyin` | 拼音分词 |
| `knn` | 向量搜索（K-Nearest Neighbors） |
| `sql` | SQL 查询支持 |
| `ingest-common` | 常用 Ingest Processor |
| `ingest-geoip` | GeoIP 地理位置解析 |

> 安装插件后需**重启 Easysearch** 才能生效。

---

## 客户端兼容

Easysearch 兼容 Elasticsearch 7.10.x 协议。连接时请注意：

| 组件 | 推荐版本 |
|------|----------|
| Logstash | 7.10.2 OSS |
| Filebeat | 7.10.2 OSS |
| Elasticsearch SDK | 7.10.x |
| Kibana | 不兼容，请使用 [INFINI Console](https://infinilabs.cn/docs/latest/console/) |

如果客户端版本校验失败，请在 `config/easysearch.yml` 中启用 API 兼容模式：

```yaml
elasticsearch.api_compatibility: true
```
