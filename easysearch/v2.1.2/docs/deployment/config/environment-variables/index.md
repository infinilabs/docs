---
title: "环境变量参考"
date: 0001-01-01
summary: "环境变量参考 #  Easysearch 支持通过环境变量控制启动行为、JVM 设置和运行路径等。本页提供所有支持的环境变量的完整参考。
核心环境变量 #     变量 说明 默认值     ES_HOME Easysearch 安装根目录 从启动脚本位置自动推断   ES_PATH_CONF 配置文件目录路径 $ES_HOME/config   ES_JAVA_HOME 自定义 Java 安装路径（推荐使用此变量） 未设置（优先使用内置 JDK）   JAVA_HOME Java 安装路径（后备方案，ES_JAVA_HOME 优先） 系统默认   ES_JAVA_OPTS 附加 JVM 选项，如堆大小 空   ES_TMPDIR 临时文件目录 自动创建   ES_STARTUP_SLEEP_TIME 后台启动后的等待时间（秒） 未设置    JDK 选择优先级 #  Easysearch 按以下优先级选择 Java 运行环境："
---


# 环境变量参考

Easysearch 支持通过环境变量控制启动行为、JVM 设置和运行路径等。本页提供所有支持的环境变量的完整参考。

## 核心环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `ES_HOME` | Easysearch 安装根目录 | 从启动脚本位置自动推断 |
| `ES_PATH_CONF` | 配置文件目录路径 | `$ES_HOME/config` |
| `ES_JAVA_HOME` | 自定义 Java 安装路径（推荐使用此变量） | 未设置（优先使用内置 JDK） |
| `JAVA_HOME` | Java 安装路径（后备方案，`ES_JAVA_HOME` 优先） | 系统默认 |
| `ES_JAVA_OPTS` | 附加 JVM 选项，如堆大小 | 空 |
| `ES_TMPDIR` | 临时文件目录 | 自动创建 |
| `ES_STARTUP_SLEEP_TIME` | 后台启动后的等待时间（秒） | 未设置 |

## JDK 选择优先级

Easysearch 按以下优先级选择 Java 运行环境：

```
1. 内置 JDK（$ES_HOME/jdk）       ← 最优先，推荐使用
2. ES_JAVA_HOME 环境变量          ← 无内置 JDK 时使用
3. JAVA_HOME 环境变量             ← 最终后备方案
```

> **建议**：使用 Easysearch 自带的内置 JDK，除非有特殊需求需要使用其他 Java 版本。

## 路径相关

### ES_PATH_CONF

指定配置文件目录，该目录下需包含 `easysearch.yml`、`jvm.options`、`log4j2.properties` 等配置文件。

```bash
# 使用自定义配置目录
export ES_PATH_CONF=/etc/easysearch
bin/easysearch
```

或在启动命令中内联设置：

```bash
ES_PATH_CONF=/opt/config bin/easysearch -d
```

### ES_TMPDIR

Easysearch 启动时会在 `$ES_TMPDIR`（或系统临时目录）下创建一个专用的临时目录。如果需要指定临时文件位置：

```bash
export ES_TMPDIR=/data/easysearch/tmp
```

## JVM 相关

### ES_JAVA_OPTS

覆盖或追加 JVM 选项。最常见的用途是设置堆大小：

```bash
export ES_JAVA_OPTS="-Xms8g -Xmx8g"
bin/easysearch
```

> **注意**：`ES_JAVA_OPTS` 中的值会追加到 `jvm.options` 文件中的设置之后，因此可以覆盖 `jvm.options` 中的同名选项。

### 被忽略的变量

以下 Java 相关环境变量会被 Easysearch 启动脚本**主动忽略**，并输出警告信息：

| 变量 | 处理方式 | 替代方案 |
|------|---------|---------|
| `JAVA_TOOL_OPTIONS` | **取消设置**，输出警告 | 使用 `ES_JAVA_OPTS` |
| `JAVA_OPTS` | **忽略**，输出警告 | 使用 `ES_JAVA_OPTS` |

## 在配置文件中引用环境变量

`easysearch.yml` 支持使用 `${...}` 语法引用环境变量的值：

```yaml
# 使用环境变量设置集群名称
cluster.name: ${CLUSTER_NAME}

# 使用环境变量设置网络地址
network.host: ${ES_NETWORK_HOST}

# 带默认值（如果环境变量未设置，使用默认值）
node.name: ${HOSTNAME:my-node}
```

使用示例：

```bash
export CLUSTER_NAME=production
export ES_NETWORK_HOST=0.0.0.0
bin/easysearch
```

> **提示**：任何自定义环境变量都可以在 `easysearch.yml` 中通过 `${VAR_NAME}` 引用，不限于以 `ES_` 开头的变量。

## Docker 环境特有变量

在 Docker 部署模式下，Easysearch 支持一些额外的环境变量和特殊行为：

| 变量 | 说明 |
|------|------|
| `EASYSEARCH_INITIAL_ADMIN_PASSWORD` | 初始管理员密码 |
| `ELASTIC_PASSWORD` | elastic 用户密码 |
| `ELASTIC_PASSWORD_FILE` | 从文件读取 elastic 密码（与 `ELASTIC_PASSWORD` 互斥） |
| `KEYSTORE_PASSWORD` | Keystore 密码 |
| `KEYSTORE_PASSWORD_FILE` | 从文件读取 keystore 密码（与 `KEYSTORE_PASSWORD` 互斥） |

### Docker 中的自动设置转换

在 Docker 部署中，**任何包含点号的环境变量**会自动转换为 `-E` 命令行参数：

```bash
# Docker 环境变量
cluster.name=my_cluster
network.host=0.0.0.0
discovery.type=single-node

# 自动转换为启动参数
bin/easysearch -Ecluster.name=my_cluster -Enetwork.host=0.0.0.0 -Ediscovery.type=single-node
```

Docker Compose 示例：

```yaml
services:
  easysearch:
    image: infinilabs/easysearch:latest
    environment:
      - cluster.name=my_cluster
      - node.name=node-1
      - discovery.type=single-node
      - EASYSEARCH_INITIAL_ADMIN_PASSWORD=my_password
      - ES_JAVA_OPTS=-Xms4g -Xmx4g
```

### `_FILE` 后缀变量

支持 `_FILE` 后缀的变量用于从文件读取敏感信息（配合 Docker Secrets 使用）：

```yaml
services:
  easysearch:
    environment:
      - ELASTIC_PASSWORD_FILE=/run/secrets/elastic_password
    secrets:
      - elastic_password
secrets:
  elastic_password:
    file: ./elastic_password.txt
```

> **要求**：`_FILE` 指向的文件权限必须为 `400` 或 `600`，且与非 `_FILE` 变体互斥。

## 系统属性

Easysearch 启动时会自动设置以下 Java 系统属性：

| 系统属性 | 来源 | 说明 |
|---------|------|------|
| `es.path.home` | `$ES_HOME` | 安装根目录 |
| `es.path.conf` | `$ES_PATH_CONF` | 配置目录 |
| `es.distribution.flavor` | 构建时确定 | 发行版类型 |
| `es.distribution.type` | 构建时确定 | 安装方式（tar/docker/rpm） |
| `es.bundled_jdk` | 构建时确定 | 是否包含内置 JDK |

## 架构相关行为

| 架构 | 特殊行为 |
|------|---------|
| `arm64` / `aarch64` | JVM 启动参数包含 `-Xshare:off`（禁用 CDS） |
| 其他架构 | JVM 使用 `-Xshare:auto`（自动使用 CDS，映射失败时回退） |

## 配置方式总览

Easysearch 支持多种配置方式，适用于不同场景：

| 方式 | 适用场景 | 持久性 | 安全性 |
|------|---------|--------|--------|
| `easysearch.yml` | 固定配置（路径、网络、发现） | ✅ 持久 | ⚠️ 明文 |
| `-E` 命令行参数 | 临时覆盖、测试 | ❌ 临时 | ⚠️ 进程列表可见 |
| 环境变量 + `${...}` | 容器化部署、不同环境差异化 | ❌ 临时 | ⚠️ 明文 |
| [Keystore]({{< relref "keystore" >}}) | 敏感信息（密码、密钥） | ✅ 持久 | ✅ 加密 |
| `jvm.options` / `jvm.options.d/` | JVM 参数 | ✅ 持久 | N/A |
| `_cluster/settings` API | 动态集群设置 | ✅/❌ | N/A |

