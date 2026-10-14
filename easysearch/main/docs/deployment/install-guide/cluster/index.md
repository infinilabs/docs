---
title: "分布式集群"
date: 0001-01-01
summary: "分布式集群安装 #  本文档介绍如何使用 initialize-cluster.sh 脚本快速搭建 Easysearch 分布式集群。该脚本支持伪分布式（单机多节点）和真分布式（多服务器）两种部署模式。
 要求 Easysearch 版本 &gt;= 2.0.3。
 前置要求 #   已完成系统调优 推荐使用 Easysearch 2.5.0 Bundle 包内置的 JDK 25；no-JDK 包请预先准备 JDK 25。JDK 21 仅用于兼容已有客户环境，见安装指南 已安装 OpenSSL 真分布式部署需要：  服务器间网络互通 自动分发时，初始化主机已配置到各目标服务器的 SSH 无密码登录 目标路径有写入权限 已创建用于运行服务的非 root 用户（本文使用 easysearch），Easysearch 不支持以 root 启动     说明：
 新部署推荐使用内置 JDK 25 的 Bundle 包，或预先准备 JDK 25；脚本在需要下载 JDK 时默认下载 JDK 25。 本地伪分布式 (--nodes N 或全部 127."
---


# 分布式集群安装

本文档介绍如何使用 `initialize-cluster.sh` 脚本快速搭建 Easysearch 分布式集群。该脚本支持伪分布式（单机多节点）和真分布式（多服务器）两种部署模式。

> 要求 Easysearch 版本 >= 2.0.3。

## 前置要求

1. 已完成[系统调优]({{< relref "/docs/deployment/config/settings" >}})
2. 推荐使用 Easysearch 2.5.0 Bundle 包内置的 JDK 25；no-JDK 包请预先准备 JDK 25。JDK 21 仅用于兼容已有客户环境，见[安装指南]({{< relref "./_index.md" >}})
3. 已安装 OpenSSL
4. 真分布式部署需要：
   - 服务器间网络互通
   - 自动分发时，初始化主机已配置到各目标服务器的 SSH 无密码登录
   - 目标路径有写入权限
   - 已创建用于运行服务的非 root 用户（本文使用 `easysearch`），Easysearch 不支持以 root 启动

> 说明：
> - 新部署推荐使用内置 JDK 25 的 Bundle 包，或预先准备 JDK 25；脚本在需要下载 JDK 时默认下载 JDK 25。
> - 本地伪分布式 (`--nodes N` 或全部 `127.0.0.1`) 在缺失 `jdk/` 且使用 `-s/--silent` 时，会自动下载 JDK。
> - 真分布式 `-s/--silent` 不会自动为目标节点选择架构并下载 JDK；请预先准备 Java，或使用交互模式选择目标架构后下载 bundled JDK。
> - 分布式节点的 JDK 必须匹配目标服务器的操作系统与架构；初始化主机也需要可用 Java 来生成密码。
> - 初始化前应在源发行包中准备好所需插件，推荐直接使用 Bundle 包；脚本先复制节点目录，随后安装的插件不会自动同步到已复制的节点。Bundle 已含所需插件时，在重装提示处选择 `N`。

## 快速开始

### 交互式安装（推荐）

交互式模式会引导您输入集群配置信息：

```bash
cd /data/easysearch
bin/initialize-cluster.sh
```

脚本会依次提示您输入：
- **集群名称**：默认为 `easysearch-cluster`
- **证书域名**：默认为 `infini.cloud`
- **API 兼容性**：是否启用 Elasticsearch API 兼容模式
- **节点数量**：默认为 `3`
- **每个节点的配置**：IP 地址、HTTP 端口、节点名称、节点角色
- **初始化确认**，以及 JDK、插件安装和 SSH 自动分发的相关提示

例如，三台不同服务器可分别输入 IP `192.168.1.10`、`192.168.1.11`、`192.168.1.12`，HTTP 端口均为 `9200`，节点名分别为 `node1`、`node2`、`node3`。默认每个节点具有 `master,data,ingest,remote_cluster_client` 角色；同一 IP 的多个节点才会默认递增 HTTP 端口。

HTTP/HTTPS 和证书信息通过 `--http`、`--cert-subject`、`--cert-days` 参数指定，没有单独的交互提示。

> **提示**：配置输入可回车使用默认值；初始化及自动分发确认需按提示输入 `y`。

### 静默安装

#### 本地伪分布式（3 节点）

在同一台机器上运行 3 个完整的 Easysearch 实例，自动分配不同端口：

```bash
cd /data/easysearch
bin/initialize-cluster.sh -s
```

若当前发行包目录下没有 `jdk/`，脚本会在静默模式下自动下载适用于当前宿主机架构的 JDK，再继续初始化。

这将创建 `/data/3nodes/` 目录，包含：
- **easysearch1**: 127.0.0.1:9200 (transport: 9300)
- **easysearch2**: 127.0.0.1:9201 (transport: 9301)
- **easysearch3**: 127.0.0.1:9202 (transport: 9302)

每个实例都是完整的 Easysearch 副本（包含 bin、lib、config、plugins 等）。

#### 真实分布式（3 台服务器）

为 3 台服务器静默生成集群配置，随后按[手动部署](#手动部署)步骤打包和分发：

```bash
cd /data/easysearch
bin/initialize-cluster.sh -s \
  -n prod-cluster \
  -i 192.168.64.3,192.168.64.4,192.168.64.5
```

脚本会在 `/data/cluster-prod-cluster/` 生成每个节点的完整配置、证书和包含密码的 `README.md`。`-s` 模式不会自动 SSH 分发，也不会预先生成供手动分发的压缩包。

如需脚本自动 SSH 分发，去掉 `-s`，并在分发提示处输入 `y`。

#### 自定义节点配置

通过 IP、HTTP 端口和节点名列表指定节点：

```bash
cd /data/easysearch
bin/initialize-cluster.sh -s \
  -n my-cluster \
  --ips 192.168.1.10,192.168.1.11,192.168.1.12 \
  --ports 9200,9200,9200 \
  --names node1,node2,node3
```

三组列表的元素数量必须一致。`-n` 仅用于集群名，脚本没有 `--node` 或命令行角色参数；需要分别指定角色时，使用不带节点列表的交互模式，或在启动前编辑各节点的 `node.roles`。`cluster.initial_master_nodes` 应只列出具有 `master` 角色的节点。

#### 自定义证书信息

指定证书的组织信息（示例使用通用占位符）：

```bash
cd /data/easysearch
bin/initialize-cluster.sh -s \
  -i 192.168.1.10,192.168.1.11,192.168.1.12 \
  --cert-subject "/C=IN/ST=FI/L=NI/O=ORG/OU=UNIT/CN="
```

生成的证书 DN 将是：
```
CN=node1.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN
```

> **注意**：默认使用通用占位符（IN/FI/NI）以避免暴露真实地理位置信息。生产环境可根据实际需求自定义。

**自定义证书有效期**：

```bash
# 生成有效期为 20 年的证书
bin/initialize-cluster.sh -s \
  -i 192.168.1.10,192.168.1.11,192.168.1.12 \
  --cert-days 7300
```

#### 启用 Elasticsearch API 兼容性

```bash
cd /data/easysearch
bin/initialize-cluster.sh -s \
  -i 192.168.1.10,192.168.1.11,192.168.1.12 \
  --enable-es-compat 8.9.0
```

将在配置中添加：
```yaml
elasticsearch.api_compatibility: true
elasticsearch.api_compatibility_version: "8.9.0"
```

对于 Elasticsearch 8.19 客户端（包括 HNSW 向量客户端协议），使用经过验证的版本标识：

```bash
cd /data/easysearch
bin/initialize-cluster.sh -s \
  -i 192.168.1.10,192.168.1.11,192.168.1.12 \
  --enable-es-compat 8.19.17
```

兼容模式只覆盖 Easysearch 已实现的部分 Elasticsearch API，不表示全部 Elasticsearch 8.x API 均可用。初始化或修改
兼容设置后需要重启对应节点；集群设置接口会拒绝修改这两个静态配置。

#### 使用 HTTP 协议（关闭 HTTP TLS）

```bash
cd /data/easysearch
bin/initialize-cluster.sh -s \
  -i 192.168.1.10,192.168.1.11,192.168.1.12 \
  --http
```

该参数只关闭 HTTP 接口的 TLS，节点间 Transport TLS 仍保留。

## TLCP 集群初始化（当前脚本）

本节介绍 `bin/initialize-cluster.sh --tlcp` 的使用方式。

### TLCP 常用参数

- `--tlcp`：启用 TLCP 双证书初始化
- `--nodes <COUNT>`：快速本地多节点（`127.0.0.1`）
- `--ips`、`--ports`、`--names`：显式指定节点 IP/HTTP 端口/节点名
- `-n, --cluster-name`：集群名
- `-d, --domain`：证书域名
- `--cert-days`、`--cert-subject`：证书有效期与 Subject
- `--enable-es-compat <VERSION>`：开启 ES 兼容模式
- `-s, --silent`、`-y, --yes`：静默/跳过确认

### Linux 宿主机安装 Tongsuo（推荐）

`--tlcp` 在 Linux 下需要可用的国密加密命令。

- 伪分布式（如 `--nodes 2` 或全部 `127.0.0.1`）：脚本会自动检测，缺失时自动调用 `bin/install-tongsuo-local.sh` 安装到当前用户目录。
- 真分布式（多台机器）：请在初始化主机及每台目标机器提前安装并验证 `tongsuo`。

如需手工预安装，可在 Easysearch 发行包目录执行：

```bash
chmod +x bin/install-tongsuo-local.sh
bin/install-tongsuo-local.sh --version 8.4.0

source ~/.tongsuo-switch.sh
use_tongsuo
tongsuo version
```

> `install-tongsuo-local.sh` 需要非 root 用户执行，默认安装到 `$HOME/local/tongsuo`，不覆盖系统 OpenSSL。

可选参数示例：

```bash
# 自定义安装路径
bin/install-tongsuo-local.sh --version 8.4.0 --prefix "$HOME/local/tongsuo"

# 离线安装（本地 tar.gz）
bin/install-tongsuo-local.sh --archive-file /path/to/tongsuo-8.4.0.tar.gz
```

### TLCP 快速示例

```bash
# 2 节点本地 TLCP 集群
cd /data/easysearch
bin/initialize-cluster.sh --nodes 2 --tlcp -y
```

默认会生成目录：

```bash
/data/easysearch/../2nodes
```

```bash
# 3 节点本地 TLCP 集群（显式节点信息）
cd /data/easysearch
bin/initialize-cluster.sh --tlcp \
  --ips 127.0.0.1,127.0.0.1,127.0.0.1 \
  --ports 9200,9201,9202 \
  --names node1,node2,node3 \
  -n tlcp-local \
  -y
```

```bash
# 2 节点真分布式 TLCP 集群（产物目录：../cluster-<cluster-name>）
cd /data/easysearch
bin/initialize-cluster.sh --tlcp \
  -n prod-tlcp \
  --ips 192.168.64.3,192.168.64.4 \
  --ports 9200,9200 \
  --names node1,node2 \
  -y
```

真分布式 TLCP 同样需要按下文部署步骤分发，并在启动前对齐 `transport.port` 与 `discovery.seed_hosts`；本例两台不同服务器均使用9300。

### TLCP 产物与启动验证

每个节点 `config/` 下会包含 TLCP 相关证书：
- `instance_sign.crt` / `instance_sign.key`
- `instance_enc.crt` / `instance_enc.key`
- `client_sign.crt` / `client_sign.key`
- `client_enc.crt` / `client_enc.key`
- `ca_chain.crt`
- `admin.crt` / `admin.key`
- `tongsuo-env.sh`（Linux 伪分布式自动生成，用于节点启动时固定 Tongsuo 环境）

本地伪分布式可直接启动：

```bash
cd /data/easysearch/../2nodes
./start-all.sh
```

`start-all.sh` 会在启动时自动：

- 优先加载每个节点的 `config/tongsuo-env.sh`（若存在）
- 探测 Java 并回填 `JAVA_HOME`/`ES_JAVA_HOME`（优先级：`ES_JAVA_HOME` > `JAVA_HOME` > macOS 的 `java_home` > PATH 中 `java`）

验证集群：

```bash
cd easysearch1
bin/tlcp-curl.sh 'https://127.0.0.1:9200/_cluster/health?pretty' -u 'admin:YOUR_PASSWORD'
bin/tlcp-curl.sh 'https://127.0.0.1:9200/_cat/nodes?v' -u 'admin:YOUR_PASSWORD'
```

> `tlcp-curl.sh` 支持 `--url <url>` 或位置参数 `<url>`（二选一）。

默认情况下（`security.ssl.http.clientauth_mode` 未设置为 `REQUIRE`），HTTP 访问只需用户名密码，无需客户端证书。

如需显式双证书访问：

```bash
bin/tlcp-curl.sh 'https://127.0.0.1:9200/_security/sslinfo?pretty' \
  -u 'admin:YOUR_PASSWORD' \
  --cert config/client_sign.crt --key config/client_sign.key \
  --enc-cert config/client_enc.crt --enc-key config/client_enc.key \
  --ca config/ca_chain.crt
```

使用 `admin.crt` / `admin.key` 访问：

```bash
bin/tlcp-curl.sh 'https://127.0.0.1:9200/_cluster/health?pretty' \
  -u 'admin:YOUR_PASSWORD' \
  --cert config/admin.crt \
  --key config/admin.key
```

> 仅当服务端配置 `security.ssl.http.clientauth_mode: REQUIRE` 时，上述证书访问方式才是必需的。

### TLCP 模式重置 admin 密码

当 `security.ssl.use_tongsuo: true` 时，`bin/reset_admin_password.sh` 会自动切换到 TLCP 调用链（`bin/tlcp-curl.sh`）：

```bash
bin/reset_admin_password.sh
# 或指定地址
bin/reset_admin_password.sh 'https://127.0.0.1:9200'
```

### TLCP 注意事项

- `--tlcp` 与 `--http` 互斥，不能同时使用
- Linux 真分布式部署需确保初始化主机和目标机器都已准备好可用 `tongsuo`；伪分布式缺失时脚本会自动安装到当前非 root 用户目录
- `start-all.sh` / `stop-all.sh` 仅在伪分布式模式生成

## 命令行参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-h, --help` | 显示帮助信息 | `--help` |
| `-s, --silent` | 静默模式，使用默认值；本地伪分布式缺失 bundled JDK 时会自动下载 | `-s` |
| `-y, --yes` | 跳过生成配置前的汇总确认 | `-y` |
| `-n, --cluster-name` | 集群名称 | `-n prod-cluster` |
| `-d, --domain` | 证书域名 | `-d example.com` |
| `-i, --ips` | IP 列表（逗号分隔）；未指定端口和节点名时，各 HTTP 端口默认为9200，节点名依次为node1、node2等 | `-i 192.168.1.10,192.168.1.11` |
| `-p, --ports` | HTTP 端口列表（逗号分隔） | `-p 9200,9201` |
| `--names` | 节点名称列表（逗号分隔） | `--names node1,node2` |
| `--nodes` | 本地多节点，HTTP 从9200、Transport 从9300递增 | `--nodes 3` |
| `--http` | 关闭 HTTP TLS，默认使用 HTTPS | `--http` |
| `--tlcp` | 生成 TLCP 双证书；与 `--http` 互斥 | `--tlcp` |
| `--cert-subject` | 证书 Subject 前缀，应以 `CN=` 结尾 | `--cert-subject "/C=CN/ST=Hunan/L=Changsha/O=MyOrg/OU=IT/CN="` |
| `--cert-days` | 证书有效期（天）| `--cert-days 7300` (20年) |
| `--enable-es-compat` | 启用 Elasticsearch API 兼容性并指定版本 | `--enable-es-compat 8.9.0` |

`-y` 不会跳过已有目录删除、JDK、插件或 SSH 分发的交互确认。需要静默生成配置时使用 `-s`；输出目录已存在时，仍会要求确认是否删除。

## 生成的目录结构

### 伪分布式模式

脚本会在当前 Easysearch 的父目录创建集群目录：

```
/data/
└── 3nodes/                           # 节点数量命名
    ├── easysearch1/                  # 完整的 Easysearch 实例 1
    │   ├── bin/
    │   ├── lib/
    │   ├── modules/
    │   ├── plugins/                  # 已安装插件
    │   ├── jdk/                      # JDK（如果下载）
    │   ├── config/
    │   │   ├── easysearch.yml       # 节点 1 配置
    │   │   ├── jvm.options
    │   │   ├── log4j2.properties
    │   │   ├── instance.crt         # 节点 1 证书
    │   │   ├── instance.key
    │   │   ├── ca.crt
    │   │   ├── admin.crt
    │   │   ├── admin.key
    │   │   └── security/
    │   │       └── user.yml          # 加密后的密码
    │   ├── data/
    │   └── logs/
    ├── easysearch2/                  # 完整的 Easysearch 实例 2
    │   └── ...
    ├── easysearch3/                  # 完整的 Easysearch 实例 3
    │   └── ...
    ├── start-all.sh                  # 启动所有节点
    ├── stop-all.sh                   # 停止所有节点
    └── README.md                     # 集群信息（含密码）
```

### 真分布式模式

以安装目录 `/data/easysearch`、集群名 `prod-cluster` 为例，脚本创建以下节点目录。只有非静默模式下确认自动分发，才会通过 SSH 部署：

```
/data/
└── cluster-prod-cluster/             # 集群配置目录
    ├── 192-168-64-3/                # 节点 1 包（待分发）
    │   ├── bin/
    │   ├── lib/
    │   ├── config/
    │   │   ├── easysearch.yml       # network.host: 192.168.64.3
    │   │   └── ...
    │   └── ...
    ├── 192-168-64-4/                # 节点 2 包
    │   └── ...
    ├── 192-168-64-5/                # 节点 3 包
    │   └── ...
    └── README.md                     # 部署说明和管理员密码
```

证书生成临时目录不保留在该目录下。手动部署需自行打包；自动部署中的压缩包是临时文件，不能依赖其在部署后继续存在。

分发到目标服务器后的结构（每台服务器）：

```
/data/easysearch/                     # 统一目录名
├── bin/
├── lib/
├── config/
│   ├── easysearch.yml               # 包含对应 IP 的配置
│   └── ...
├── data/
├── logs/
└── pid                               # 启动后生成
```

## 部署方式

### 方式一：本地伪分布式

适合开发测试环境，所有节点运行在同一台机器上。

1. **生成集群**

```bash
cd /data/easysearch
bin/initialize-cluster.sh -s
```

2. **启动所有节点**

```bash
cd /data/3nodes
./start-all.sh
```

3. **验证集群**

```bash
# 获取管理员密码
grep -F 'Admin Password' README.md

# 检查集群健康状态
curl -ku 'admin:YOUR_PASSWORD' "https://127.0.0.1:9200/_cluster/health?pretty"

# 查看节点列表
curl -ku 'admin:YOUR_PASSWORD' "https://127.0.0.1:9200/_cat/nodes?v"
```

输出示例：
```
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           45          68   8    0.52    0.38     0.35 dimr      *      node1
127.0.0.1           38          68   8    0.52    0.38     0.35 dimr      -      node2
127.0.0.1           42          68   8    0.52    0.38     0.35 dimr      -      node3
```

4. **停止集群**

```bash
cd /data/3nodes
./stop-all.sh
```

> **说明**：`stop-all.sh` 脚本会读取每个节点目录下的 `pid` 文件来优雅地停止进程。

### 方式二：真实分布式部署

适合生产环境，节点分布在多台服务器上。脚本支持自动通过 SSH 分发。

#### 自动部署（推荐）

**前提条件**：
- 当前主机已与目标服务器建立 SSH 互信
- 目标服务器的 `/data` 目录有写入权限

自动部署会替换目标服务器的 `/data/easysearch`，仅用于已确认可替换的安装目录；已有服务需先停止并备份数据。

**步骤 1：生成并自动部署**

```bash
cd /data/easysearch
bin/initialize-cluster.sh \
  -n prod-cluster \
  -i 192.168.64.3,192.168.64.4,192.168.64.5
```

在 `Automatically deploy to remote servers via SSH?` 提示处输入 `y`。保留 `-s` 时只会生成配置。

脚本会自动：
1. 生成所有节点的配置和证书
2. 为每个节点创建完整的 Easysearch 实例
3. 打包并通过 SSH 复制到对应服务器
4. 在远程服务器上自动解压到 `/data/easysearch`

**步骤 2：在各服务器上启动节点**

启动前，在各服务器上将 `config/easysearch.yml` 中的 `transport.port` 配置为 `9300`，与生成的 `discovery.seed_hosts` 对齐。当前脚本会给节点依次写入9300、9301、9302，但分布式发现地址均使用9300，因此需要完成这一步；不要在同一台服务器的多节点场景中统一端口。

在每台服务器上准备目录权限，并以非 root 用户启动：

```bash
cd /data/easysearch
sudo chown -R easysearch:easysearch /data/easysearch
sudo -u easysearch bin/easysearch -d -p pid
```

或者在管理节点上通过 SSH 批量启动：

```bash
for ip in 192.168.64.3 192.168.64.4 192.168.64.5; do
  ssh "$ip" 'sudo chown -R easysearch:easysearch /data/easysearch && cd /data/easysearch && sudo -u easysearch bin/easysearch -d -p pid'
done
```

#### 手动部署

如果 SSH 自动部署失败或需要手动控制，可以使用手动方式。

**步骤 1：生成集群配置**

```bash
cd /data/easysearch
bin/initialize-cluster.sh -s \
  -n prod-cluster \
  -i 192.168.64.3,192.168.64.4,192.168.64.5
```

此命令只生成 `/data/cluster-prod-cluster/` 下的节点目录。启动前需将各节点的 `transport.port` 与 `discovery.seed_hosts` 对齐；本例三台不同服务器均使用9300。若此前仅部分节点分发成功，直接使用本轮已生成的节点目录继续打包分发，保留同一套证书和密码。

**步骤 2：手动分发**

先打包各节点目录，再将压缩包复制到对应服务器：

```bash
cd /data/cluster-prod-cluster

for node in 192-168-64-3 192-168-64-4 192-168-64-5; do
  tar -czf "$node.tar.gz" "$node"
done

# 复制到服务器 1
scp 192-168-64-3.tar.gz root@192.168.64.3:/tmp/

# 复制到服务器 2
scp 192-168-64-4.tar.gz root@192.168.64.4:/tmp/

# 复制到服务器 3
scp 192-168-64-5.tar.gz root@192.168.64.5:/tmp/
```

**步骤 3：在各服务器上解压**

在每台服务器上执行：

下面以服务器1为例；服务器2、3分别替换成自己的压缩包及目录名。`/data/easysearch` 应不存在，避免 `mv` 将节点目录嵌入已有安装。

```bash
cd /tmp
tar -xzf 192-168-64-3.tar.gz
mv 192-168-64-3 /data/easysearch
chown -R easysearch:easysearch /data/easysearch
```

**步骤 4：启动节点**

```bash
cd /data/easysearch
su easysearch -c "bin/easysearch -d -p pid"
```

#### 验证集群

在任意节点上验证集群状态：

```bash
# 获取管理员密码（在生成配置的服务器上）
grep -F 'Admin Password' /data/cluster-prod-cluster/README.md

# 检查集群健康状态
curl -ku 'admin:YOUR_PASSWORD' "https://192.168.64.3:9200/_cluster/health?pretty"

# 查看所有节点
curl -ku 'admin:YOUR_PASSWORD' "https://192.168.64.3:9200/_cat/nodes?v"
```

#### 停止节点

在每台服务器上执行：

```bash
kill $(cat /data/easysearch/pid)
```

或通过 SSH 批量停止：

```bash
for ip in 192.168.64.3 192.168.64.4 192.168.64.5; do
  ssh $ip "kill \$(cat /data/easysearch/pid)"
done
```

## 节点配置说明

每个节点的 `config/easysearch.yml` 包含以下关键配置：

```yaml
# 集群名称（所有节点相同）
cluster.name: my-cluster

# 节点名称（每个节点不同）
node.name: node1
node.roles: [ master, data, ingest, remote_cluster_client ]

# 数据和日志路径
path.data: data
path.logs: logs

# 网络配置（每个节点的 IP 不同）
network.host: 192.168.64.3
http.port: 9200
transport.port: 9300

# 集群发现（所有节点相同）
discovery.seed_hosts: ["192.168.64.3:9300", "192.168.64.4:9300", "192.168.64.5:9300"]
cluster.initial_master_nodes: ["node1", "node2", "node3"]

# 安全配置（所有节点相同）
security.enabled: true
security.ssl.http.enabled: true
security.ssl.http.cert_file: instance.crt
security.ssl.http.key_file: instance.key
security.ssl.http.ca_file: ca.crt
security.ssl.transport.enabled: true
security.ssl.transport.cert_file: instance.crt
security.ssl.transport.key_file: instance.key
security.ssl.transport.ca_file: ca.crt

# 节点认证（所有节点相同）
security.nodes_dn:
  - "CN=node1.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
  - "CN=node2.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
  - "CN=node3.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"

# 管理员认证（所有节点相同）
security.authcz.admin_dn:
  - "CN=admin.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
```

> **关键配置说明**：
> - 每个节点通过 `network.host` 和 `node.name` 进行区分
> - 每个节点使用独特的证书 CN（如 `CN=node1.infini.cloud`）
> - `security.nodes_dn` 列出所有允许加入集群的节点证书 DN
> - `security.authcz.admin_dn` 定义管理员证书 DN
> - `path.data` 和 `path.logs` 使用统一的相对路径
> - 所有节点使用相同的集群发现配置
> - 此处是三台服务器统一 Transport 端口后的配置；启动前应核对实际生成配置中的端口

## 证书和节点认证

### 证书设计

脚本为集群中的每个节点生成独立的证书，使用不同的 CN（Common Name）：

**默认证书信息**（可通过命令行参数自定义）：
- **国家 (C)**：IN
- **州/省 (ST)**：FI  
- **城市 (L)**：NI
- **组织 (O)**：ORG
- **组织单位 (OU)**：UNIT
- **域名**：infini.cloud
- **有效期**：3650 天（10 年）

**生成的证书 DN**：
- **节点 1 证书**：`CN=node1.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN`
- **节点 2 证书**：`CN=node2.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN`
- **节点 3 证书**：`CN=node3.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN`
- **管理员证书**：`CN=admin.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN`

**自定义证书信息**：

使用命令行参数自定义证书的组织信息和有效期：

```bash
bin/initialize-cluster.sh -s \
  -i 192.168.1.10,192.168.1.11 \
  --cert-subject "/C=IN/ST=FI/L=NI/O=ORG/OU=UNIT/CN=" \
  --cert-days 7300 \
  --domain infini.cloud
```

将生成有效期为 20 年的证书：
```
CN=node1.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN
```

### 节点互信配置

通过 `security.nodes_dn` 配置，明确列出所有允许加入集群的节点证书 DN。这样可以：

1. **提高安全性**：只有证书 DN 在白名单中的节点才能加入集群
2. **防止恶意节点**：即使有人获得了 CA 签发的证书，如果 DN 不在白名单中也无法加入
3. **灵活扩展**：添加新节点时，需要在所有节点配置中添加新节点的 DN

### 查看证书信息

```bash
# 查看节点证书的 CN
cd /data/easysearch
openssl x509 -in config/instance.crt -noout -subject

# 查看证书的有效期
openssl x509 -in config/instance.crt -noout -dates

# 查看证书的 SAN（Subject Alternative Names）
openssl x509 -in config/instance.crt -noout -text | grep -A1 "Subject Alternative Name"

# 查看证书的完整信息
openssl x509 -in config/instance.crt -noout -text
```

**示例输出**：
```bash
# 查看主题
$ openssl x509 -in config/instance.crt -noout -subject
subject=C = IN, ST = FI, L = NI, O = ORG, OU = UNIT, CN = node1.infini.cloud

# 查看有效期
$ openssl x509 -in config/instance.crt -noout -dates
notBefore=Jan 02 06:00:00 2026 GMT
notAfter=Dec 30 06:00:00 2035 GMT
```

## 配置为系统服务

> **注意**：以下服务示例用于 TLS 真分布式部署，创建服务前应准备 `easysearch` 用户和目录权限，并停止此前手动启动的实例。TLCP 部署还需为服务账户准备 Tongsuo 环境；本地伪分布式测试环境可使用 `start-all.sh`。

在每台生产服务器上创建 systemd 服务文件，以便系统启动时自动运行。所有服务器使用相同的服务配置，通过 `config/easysearch.yml` 中的 `network.host` 和 `node.name` 来区分不同节点。

### 在每台服务器上创建服务文件

每台服务器都使用相同的服务配置（`/usr/lib/systemd/system/easysearch.service`）：

```bash
sudo tee /usr/lib/systemd/system/easysearch.service <<-'EOF'
[Unit]
Description=Easysearch
Documentation=https://www.infinilabs.com
After=network.target

[Service]
Type=forking
User=easysearch
WorkingDirectory=/data/easysearch
Environment="ES_PATH_CONF=/data/easysearch/config"
PIDFile=/data/easysearch/pid
ExecStart=/data/easysearch/bin/easysearch -d -p pid
PrivateTmp=true
LimitNOFILE=65536
LimitNPROC=65536
LimitAS=infinity
LimitFSIZE=infinity
LimitMEMLOCK=infinity
TimeoutStopSec=0
KillSignal=SIGTERM
KillMode=process
SendSIGKILL=no
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
EOF
```

> **说明**：
> - 所有服务器使用统一的服务名：`easysearch.service`
> - 工作目录统一为：`/data/easysearch`
> - 节点通过配置文件 `/data/easysearch/config/easysearch.yml` 中的 IP 和节点名区分
> - 不需要使用 `node1`、`node2` 等子目录

### 启用并启动服务

在每台服务器上执行：

```bash
# 重新加载服务配置
sudo systemctl daemon-reload

# 启用开机自启
sudo systemctl enable easysearch

# 启动服务
sudo systemctl start easysearch

# 查看状态
sudo systemctl status easysearch
```

## 常见问题

### 1. 节点无法加入集群

**现象**：节点启动正常，但集群中只显示一个节点。

**排查步骤**：
1. 检查网络连通性：`telnet 192.168.1.10 9300`
2. 检查防火墙：确保 HTTP 端口和 Transport 端口已开放
3. 检查日志：查看 `logs/easysearch.log` 中的错误信息
4. 检查集群名称：确保所有节点的 `cluster.name` 相同

### 2. 证书验证失败

**现象**：节点日志中出现证书错误。

**解决方案**：
- 确保所有节点使用相同的 CA 证书（`ca.crt`）
- 当前 TLS 脚本生成的 SAN 为证书域名及其通配域名，不包含节点 IP；默认 Transport 配置使用 `security.ssl.transport.skip_domain_verify: true`
- 如需开启节点主机名验证，使用覆盖实际连接地址的证书 SAN，并核对对应验证配置；直接通过 IP 访问 HTTPS 时也需要匹配的 IP SAN

### 3. 端口冲突

**现象**：节点启动失败，提示端口被占用。

**解决方案**：
- 检查端口占用：`netstat -tuln | grep 9200`
- 使用 `--ports` 显式指定各节点 HTTP 端口；Transport 端口需在各节点的 `config/easysearch.yml` 中修改，并同步对应的 `discovery.seed_hosts`

### 4. 内存不足

**现象**：节点频繁重启或响应缓慢。

**解决方案**：
- 调整 JVM 内存：编辑 `config/jvm.options`
- 建议设置为物理内存的 50%，但不超过 32GB

```bash
# 例如设置为 4GB
-Xms4g
-Xmx4g
```

## 生产环境最佳实践

1. **节点规划**
   - 至少 3 个 master 节点（奇数个）
   - 根据数据量规划 data 节点数量
   - 考虑独立的 coordinating 节点处理查询

2. **硬件配置**
   - CPU：8 核及以上
   - 内存：32GB 及以上
   - 磁盘：SSD，至少 500GB
   - 网络：千兆网卡

3. **安全配置**
   - 修改默认管理员密码
   - 配置防火墙规则
   - 定期更新证书
   - 启用审计日志

4. **监控和备份**
   - 使用 [INFINI Console](https://infinilabs.cn/docs/latest/console/) 进行集群监控
   - 配置快照和恢复策略
   - 定期备份配置文件和证书

5. **性能优化**
   - 根据负载调整刷新间隔
   - 合理设置分片和副本数量
   - 使用 SSD 存储
   - 禁用 swap

## 更多资源

- [系统调优]({{< relref "/docs/deployment/config/settings" >}})
- [节点配置]({{< relref "/docs/deployment/config/node-settings/" >}})
- [Docker Compose 集群部署](./docker-compose.md)
- [Kubernetes 部署](./helm.md)
- [INFINI Console 文档](https://infinilabs.cn/docs/latest/console/)

## 总结

使用 `initialize-cluster.sh` 脚本可以快速搭建 Easysearch 分布式集群，支持：

- ✅ 交互式和静默两种安装模式
- ✅ 伪分布式和真分布式部署
- ✅ 自动生成节点配置和证书
- ✅ 独立的数据和日志目录
- ✅ 便捷的集群管理脚本

对于生产环境，建议使用真实分布式部署，并配置为系统服务以确保高可用性。

