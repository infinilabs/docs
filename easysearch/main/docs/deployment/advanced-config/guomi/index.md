---
title: "国密配置"
date: 0001-01-01
summary: "国密 / TLCP 配置指南 #  在政务、金融等信创场景中，要求使用国密（SM）系列算法进行 TLS 加密。Easysearch 基于 铜锁（Tongsuo） 提供国密 TLCP 双证书能力，支持 SM2/SM3/SM4 加密套件。
国密算法简介 #     算法 类型 对应国际算法 用途     SM2 非对称加密 ECDSA / RSA 数字签名、密钥交换   SM3 哈希 SHA-256 完整性校验   SM4 对称加密 AES 数据加密    国密 TLS 使用 SM2 证书 + SM4 加密 + SM3 哈希，组成完整的加密通信套件。
 前置条件 #     条件 说明     节点运行平台 国密运行依赖 Tongsuo JNI 原生库；bin/easysearch 当前会按以下平台加载本地库：linux-x86_64 / linux-aarch64 / darwin-x86_64 / darwin-aarch64。若对应库缺失会启动失败   证书生成平台 generate-tlcp-certs."
---


# 国密 / TLCP 配置指南

在政务、金融等信创场景中，要求使用国密（SM）系列算法进行 TLS 加密。Easysearch 基于 [铜锁（Tongsuo）](https://release.infinilabs.com/easysearch/archive/tongsuo/) 提供国密 TLCP 双证书能力，支持 SM2/SM3/SM4 加密套件。

## 国密算法简介

| 算法 | 类型 | 对应国际算法 | 用途 |
|------|------|-------------|------|
| **SM2** | 非对称加密 | ECDSA / RSA | 数字签名、密钥交换 |
| **SM3** | 哈希 | SHA-256 | 完整性校验 |
| **SM4** | 对称加密 | AES | 数据加密 |

国密 TLS 使用 SM2 证书 + SM4 加密 + SM3 哈希，组成完整的加密通信套件。

---

## 前置条件

| 条件 | 说明 |
|------|------|
| **节点运行平台** | 国密运行依赖 Tongsuo JNI 原生库；`bin/easysearch` 当前会按以下平台加载本地库：`linux-x86_64` / `linux-aarch64` / `darwin-x86_64` / `darwin-aarch64`。若对应库缺失会启动失败 |
| **证书生成平台** | `generate-tlcp-certs.sh` 在 Linux 上要求系统安装 `tongsuo` 命令；macOS/其他平台优先使用 `tongsuo`，未安装时 fallback 到 `openssl`（需支持 SM2/SM3）；若两者都不存在则脚本直接失败退出 |
| **安全模块** | 需使用包含 `modules/security` 的发行包 |
| **协议** | 推荐使用 `TLSv1.3`；如手工配置 `TLCP` 但运行环境不支持，会在启动时报错（不会自动回退） |

---

## Linux 安装 Tongsuo（宿主机）

当你在 Linux 宿主机上执行 `bin/generate-tlcp-certs.sh` 时，需要系统可直接找到 `tongsuo` 命令。`bin/initialize-cluster.sh --tlcp` 在 Linux 伪分布式模式下可自动安装并切换本地 Tongsuo 环境（见下文说明）。

推荐使用发行包内置脚本安装（仅安装到当前用户目录，不替换系统 OpenSSL）：

```bash
cd /path/to/easysearch
chmod +x bin/install-tongsuo-local.sh
bin/install-tongsuo-local.sh --version 8.4.0

# 加载切换函数并切换到 tongsuo
source ~/.tongsuo-switch.sh
use_tongsuo

# 验证
command -v tongsuo
tongsuo version
```

常用可选参数：

```bash
# 自定义安装目录与源码缓存目录
bin/install-tongsuo-local.sh --version 8.4.0 \
  --prefix "$HOME/local/tongsuo" \
  --source-dir "$HOME/local/src"

# 离线安装（使用本地源码包，可选 sha256 校验）
bin/install-tongsuo-local.sh \
  --archive-file /path/to/tongsuo-8.4.0.tar.gz \
  --sha256 <sha256>
```

切回系统 OpenSSL：

```bash
source ~/.tongsuo-switch.sh
use_system_openssl
```

> `install-tongsuo-local.sh` 必须由非 root 用户执行；默认安装到 `$HOME/local/tongsuo`，并写入 `$HOME/.tongsuo-switch.sh`。该脚本不会替换系统 OpenSSL，只有执行 `use_tongsuo` 后当前 shell 才切到 Tongsuo。

`initialize-cluster.sh --tlcp` 的自动环境行为（当前实现）：

- Linux + 伪分布式（例如 `--nodes 2` 或所有节点均为 `127.0.0.1`）：
  若未检测到可用 `tongsuo`，脚本会自动调用 `bin/install-tongsuo-local.sh` 安装到当前用户目录，并在每个节点写入 `config/tongsuo-env.sh`。
- Linux + 真分布式（多台机器）：
  不会远程自动安装，需各目标节点提前准备好 `tongsuo`。
- macOS：
  不强制安装 `tongsuo`；若系统 `openssl` 具备 SM2/SM3 能力也可继续初始化。

如需手工编译安装（高级场景）：

```bash
cd /tmp
curl -fL https://release.infinilabs.com/easysearch/archive/tongsuo/8.4.0.tar.gz -o tongsuo-8.4.0.tar.gz
tar -xzf tongsuo-8.4.0.tar.gz
cd Tongsuo-8.4.0
./config shared --prefix=/opt/tongsuo --openssldir=/opt/tongsuo/ssl
make -j"$(nproc)"
sudo make install_sw
```

---

## 证书生成

Easysearch 自带证书生成脚本，在发行包根目录执行：

```bash
bin/generate-tlcp-certs.sh /path/to/output "changeit"
```

> 第一个参数为输出目录，第二个参数为证书密码，第三个参数（可选）为证书 subject（默认 `/C=IN/ST=FI/L=NI/O=ORG/OU=UNIT/CN=infini.cloud`）。也可通过 `DOMAIN` 环境变量覆盖默认域名：
>
> ```bash
> DOMAIN=my.company.com bin/generate-tlcp-certs.sh /path/to/output "changeit"
> ```
>
> 生成多节点证书时，可通过 `NODE_NAMES_STR` 传入空格分隔的节点名：
>
> ```bash
> NODE_NAMES_STR="node1 node2 node3" bin/generate-tlcp-certs.sh /path/to/output "changeit"
> ```

### 脚本输出文件

TLCP 要求**签名证书**和**加密证书**两套独立的密钥对（双证书体制），脚本会同时生成服务端和客户端的双证书：

| 文件 | 说明 |
|------|------|
| `node1_sign.crt` / `node1_sign.key`（及 `nodeX_*`） | 节点签名证书与私钥（按节点生成） |
| `node1_enc.crt` / `node1_enc.key`（及 `nodeX_*`） | 节点加密证书与私钥（按节点生成） |
| `server_sign.crt` / `server_sign.key` | 兼容文件，默认指向第一个节点签名证书副本 |
| `server_enc.crt` / `server_enc.key` | 兼容文件，默认指向第一个节点加密证书副本 |
| `admin.crt` / `admin.key` | 管理员客户端证书与私钥 |
| `client_sign.crt` / `client_sign.key` | 客户端签名证书与私钥 |
| `client_enc.crt` / `client_enc.key` | 客户端加密证书与私钥 |
| `ca.crt` / `ca.key` | 根 CA 证书与私钥 |
| `sub_ca.crt` / `sub_ca.key` | 中间 CA 证书与私钥 |
| `ca_chain.crt` | 中间 CA + 根 CA 信任链 |
| `http-truststore-sm2.p12` | HTTP 层 PKCS12 Truststore |
| `transport-truststore-sm2.p12` | Transport 层 PKCS12 Truststore |
| `http-keystore-sm2.p12` / `transport-keystore-sm2.p12` | 空 PKCS12 Keystore 占位文件（兼容用途，不含 TLCP 双证书） |

生成完成后，将输出目录中的证书文件复制到 Easysearch 的 `config/` 目录。

---

## 配置方式（推荐：PEM 模式）

TLCP 双证书配置使用 `security.ssl.<层>.tlcp.*` 前缀，分别指定签名（sign）和加密（enc）两套证书。

### HTTP 层

```yaml
# ---- 启用铜锁国密引擎 ----
security.ssl.use_tongsuo: true
security.ssl.http.enabled: true

# ---- TLCP 双证书（PEM 格式） ----
security.ssl.http.tlcp.sign_certificate: server_sign.crt
security.ssl.http.tlcp.sign_key: server_sign.key
security.ssl.http.tlcp.enc_certificate: server_enc.crt
security.ssl.http.tlcp.enc_key: server_enc.key

# ---- 信任链 ----
security.ssl.http.tlcp.trusted_ca_certificate: ca_chain.crt

# ---- TLCP 协议设置 ----
security.ssl.http.tlcp.protocol: TLSv1.3
security.ssl.http.tlcp.cipher_suites:
  - "TLS_SM4_GCM_SM3"
  - "TLS_SM4_CCM_SM3"
```

### Transport 层

```yaml
# ---- 启用铜锁国密引擎 ----
security.ssl.use_tongsuo: true
security.ssl.transport.enabled: true

# ---- TLCP 双证书（PEM 格式） ----
security.ssl.transport.tlcp.sign_certificate: server_sign.crt
security.ssl.transport.tlcp.sign_key: server_sign.key
security.ssl.transport.tlcp.enc_certificate: server_enc.crt
security.ssl.transport.tlcp.enc_key: server_enc.key

# ---- 信任链 ----
security.ssl.transport.tlcp.trusted_ca_certificate: ca_chain.crt

# ---- TLCP 协议设置 ----
security.ssl.transport.tlcp.protocol: TLSv1.3
security.ssl.transport.tlcp.cipher_suites:
  - "TLS_SM4_GCM_SM3"
  - "TLS_SM4_CCM_SM3"

# ---- 节点间通信建议 ----
security.ssl.transport.skip_domain_verify: true
security.ssl.transport.resolve_hostname: false
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `security.ssl.use_tongsuo` | 是否启用铜锁国密引擎，使用国密必须设为 `true` |
| `*.tlcp.sign_certificate` | 签名证书文件（PEM），用于数字签名 |
| `*.tlcp.sign_key` | 签名私钥文件（PEM） |
| `*.tlcp.enc_certificate` | 加密证书文件（PEM），用于密钥交换 |
| `*.tlcp.enc_key` | 加密私钥文件（PEM） |
| `*.tlcp.trusted_ca_certificate` | CA 信任链文件（PEM），需使用 `ca_chain.crt`（包含中间 CA + 根 CA） |
| `*.tlcp.protocol` | TLS 协议版本，推荐 `TLSv1.3`；如设为 `TLCP`，需确认运行环境支持，否则启动报错 |
| `*.tlcp.cipher_suites` | 国密密码套件列表（可选，默认 `TLS_SM4_GCM_SM3`） |

> 文件路径相对于 `config/` 目录。`sign_alias` / `enc_alias` 在 PEM 模式可省略（默认 `sign` / `enc`，内部仍会使用该别名装载双证书），`initialize.sh --tlcp` 会自动写入这两个值。

### 可用密码套件

| 套件名称 | 说明 |
|----------|------|
| `TLS_SM4_GCM_SM3` | SM4-GCM 加密 + SM3 哈希（推荐） |
| `TLS_SM4_CCM_SM3` | SM4-CCM 加密 + SM3 哈希 |

---

## 完整配置示例

HTTP + Transport 同时启用国密：

```yaml
# ============== 国密 TLCP 完整配置 ==============

security.enabled: true

# ---- 启用铜锁引擎 ----
security.ssl.use_tongsuo: true

# ---- HTTP 层 TLCP ----
security.ssl.http.enabled: true
security.ssl.http.tlcp.sign_certificate: server_sign.crt
security.ssl.http.tlcp.sign_key: server_sign.key
security.ssl.http.tlcp.enc_certificate: server_enc.crt
security.ssl.http.tlcp.enc_key: server_enc.key
security.ssl.http.tlcp.trusted_ca_certificate: ca_chain.crt
security.ssl.http.tlcp.protocol: TLSv1.3
security.ssl.http.tlcp.cipher_suites:
  - "TLS_SM4_GCM_SM3"
  - "TLS_SM4_CCM_SM3"

# ---- Transport 层 TLCP ----
security.ssl.transport.enabled: true
security.ssl.transport.tlcp.sign_certificate: server_sign.crt
security.ssl.transport.tlcp.sign_key: server_sign.key
security.ssl.transport.tlcp.enc_certificate: server_enc.crt
security.ssl.transport.tlcp.enc_key: server_enc.key
security.ssl.transport.tlcp.trusted_ca_certificate: ca_chain.crt
security.ssl.transport.tlcp.protocol: TLSv1.3
security.ssl.transport.tlcp.cipher_suites:
  - "TLS_SM4_GCM_SM3"
  - "TLS_SM4_CCM_SM3"
security.ssl.transport.skip_domain_verify: true
security.ssl.transport.resolve_hostname: false

# ---- 访问控制 ----
security.allow_default_init_securityindex: true
security.restapi.roles_enabled: ["superuser", "security_rest_api_access"]
```

---

## 备选：PKCS12 Truststore 模式

如果需要使用 PKCS12 格式的 Truststore（不推荐，仅用于兼容场景）：

```yaml
security.ssl.http.truststore_filepath: http-truststore-sm2.p12
security.ssl.http.truststore_password: changeit

security.ssl.transport.truststore_filepath: transport-truststore-sm2.p12
security.ssl.transport.truststore_password: changeit
```

> **注意事项**：
> - **PEM 模式为推荐路径**，PKCS12 仅用于特定兼容场景。
> - 脚本生成的 `http-keystore-sm2.p12`、`transport-keystore-sm2.p12` 为**空 Keystore 占位文件**，不包含 TLCP 双证书。
> - TLCP 双证书要求 sign/enc 两套证书；使用 PKCS12 需自行构造包含双证书与别名的 Keystore，并承担 SM2/JCA 兼容性风险。

---

## tlcp-curl 工具

Easysearch 提供 `bin/tlcp-curl.sh`，用于在国密 TLCP 环境下发起 HTTP 请求，行为类似 `curl`，但底层使用铜锁 JCA Provider 建立 TLCP 连接。

```bash
bin/tlcp-curl.sh https://<host>:9200/_cluster/health -u admin:changeit
```

> `tlcp-curl.sh` 默认不发送客户端证书。传入 `--cert/--key/--enc-cert/--enc-key` 任一参数后才启用客户端证书模式；未显式指定的签名证书/密钥会回退到 `config/admin.crt` / `config/admin.key`。仅当服务端配置 `security.ssl.http.clientauth_mode: REQUIRE` 时，客户端证书才是必需的。
>
> `tlcp-curl.sh` 会自动尝试加载 Tongsuo 环境（按顺序）：`config/tongsuo-env.sh` → 已设置的 `TONGSUO_HOME` → `~/.tongsuo-switch.sh`（`use_tongsuo`）→ `$HOME/local/tongsuo` → PATH 中现有 `tongsuo`。

### 常用参数

| 参数 | 说明 |
|------|------|
| `--url <url>` 或位置参数 `<url>` | 目标 URL（二选一） |
| `-X <method>` | HTTP 方法，默认 `GET` |
| `-u <user:pass>` | HTTP Basic 认证 |
| `-H <header>` | 自定义请求头，可多次指定 |
| `-d <data>` | 请求体字符串 |
| `--data-file <file>` | 从文件读取请求体 |
| `--cert <path>` | 客户端签名证书 PEM（仅在启用客户端证书模式时使用） |
| `--key <path>` | 客户端签名私钥 PEM（仅在启用客户端证书模式时使用） |
| `--enc-cert <path>` | 客户端加密证书 PEM（可选，支持自动推导） |
| `--enc-key <path>` | 客户端加密私钥 PEM（可选，支持自动推导） |
| `--no-cert` | 不发送客户端证书（当前默认行为） |
| `--ca <path>` | 信任 CA PEM，默认 `config/ca_chain.crt` |
| `--insecure` | 跳过服务端证书和主机名验证（当前默认行为，参数保留用于兼容） |
| `--protocol <name>` | TLS 协议，默认 `TLSv1.3` |
| `--cipher <name>` | 密码套件，默认 `TLS_SM4_GCM_SM3` |
| `-i` | 输出中包含响应头 |
| `-s` | 静默模式，不输出诊断信息 |
| `-v` | 详细调试输出 |
| `-o <file>` | 将响应体写入文件 |
| `-w <format>` | 输出格式串（仅短参数），支持 `%{http_code}`、`%{time_total}` |
| `--connect-timeout <s>` | 连接超时秒数，默认 10 |
| `--max-time <s>` | 最大等待秒数，默认 60 |

> `--enc-cert` / `--enc-key` 自动推导规则（优先级）：显式参数 > 从 `--cert/--key` 路径中的 `_sign` 推导 `_enc` > `config/client_enc.crt|key` > 回退到签名证书/密钥路径。

### 示例

验证国密端点（默认场景，无需客户端证书）：

```bash
bin/tlcp-curl.sh https://localhost:9200/_security/sslinfo?pretty \
  -u admin:changeit
```

服务端启用 `security.ssl.http.clientauth_mode: REQUIRE` 时，使用客户端双证书（双向认证）：

```bash
bin/tlcp-curl.sh https://localhost:9200/_cluster/health \
  --cert config/client_sign.crt --key config/client_sign.key \
  --enc-cert config/client_enc.crt --enc-key config/client_enc.key \
  --ca config/ca_chain.crt
```

仅使用用户名密码（不发送客户端证书）：

```bash
bin/tlcp-curl.sh https://localhost:9200/_cluster/health?pretty \
  -u admin:changeit
```

使用 `admin.crt` / `admin.key` 访问：

```bash
bin/tlcp-curl.sh https://localhost:9200/_cluster/health?pretty \
  -u admin:changeit \
  --cert config/admin.crt \
  --key config/admin.key
```

输出状态码与耗时（`-w`）：

```bash
bin/tlcp-curl.sh https://localhost:9200/_cluster/health \
  --cert config/client_sign.crt --key config/client_sign.key \
  --enc-cert config/client_enc.crt --enc-key config/client_enc.key \
  --ca config/ca_chain.crt \
  -o /tmp/tlcp-health.json \
  -w 'http=%{http_code} time=%{time_total}s\n'
```

使用 `PUT`（与 `curl` 类似）：

```bash
bin/tlcp-curl.sh https://localhost:9200/_security/user/test_user \
  -X PUT \
  -u admin:changeit \
  -H 'Content-Type: application/json' \
  -d '{"password":"Test@123","external_roles":["admin"]}'
```

> 工具依赖 `modules/security/` 下的 jar 包，需在 Easysearch 发行包根目录下执行。

### TLCP 模式重置 admin 密码

`bin/reset_admin_password.sh` 在 TLCP 模式下可直接使用，无需额外参数切换。脚本会自动识别 `security.ssl.use_tongsuo: true`，并通过 `bin/tlcp-curl.sh` + `config/admin.crt|admin.key` 调用安全 API：

```bash
bin/reset_admin_password.sh
# 或指定地址
bin/reset_admin_password.sh 'https://127.0.0.1:9200'
```

---

## 启动与验证

### 1. 启动节点

正常启动 Easysearch，日志中应能看到 Tongsuo 安全提供者加载成功。

### 2. 查看 SSL 信息

```bash
bin/tlcp-curl.sh --url https://<host>:9200/_security/sslinfo?pretty \
  -u admin:changeit \
  --cert config/client_sign.crt --key config/client_sign.key \
  --enc-cert config/client_enc.crt --enc-key config/client_enc.key \
  --ca config/ca_chain.crt
```

期望返回中包含：

```json
{
  "ssl_provider" : "Tongsuo_Security_Provider",
  "local_certificates_list" : [ ... ]
}
```

确认 `ssl_provider` 为 `Tongsuo_Security_Provider` 且 `local_certificates_list` 中包含签名和加密证书即表示配置成功。

---

## 常见问题

### `SSLContext.getInstance("TLCP")` 报错

当前运行环境的 JDK 不支持 `TLCP` 协议名，请将 `protocol` 改为 `TLSv1.3`：

```yaml
security.ssl.http.tlcp.protocol: TLSv1.3
```

铜锁 OpenJDK 在 `TLSv1.3` 下依然会使用国密套件进行协商。

### Cipher Suite 不生效

TLS 1.3 的密码套件不可通过 `setEnabledCipherSuites` 禁用，这是 TLS 1.3 协议规范的行为，不影响国密功能。

### 证书链验证失败

请确认使用 `ca_chain.crt`（包含中间 CA + 根 CA 的完整链）而不是单独的 `ca.crt`：

```yaml
# ✅ 正确 — 使用完整信任链
security.ssl.http.tlcp.trusted_ca_certificate: ca_chain.crt

# ❌ 错误 — 仅包含根 CA，缺少中间 CA
security.ssl.http.tlcp.trusted_ca_certificate: ca.crt
```

### 平台与原生库问题

可按“证书生成”和“节点运行”两个维度理解平台支持：

| 维度 | 支持平台 | 说明 |
|------|----------|------|
| **证书生成** | Linux（x86_64 / aarch64） | 必须安装并使用 `tongsuo` 命令 |
| **证书生成** | macOS（x86_64 / arm64）及其他非 Linux 平台 | 优先 `tongsuo`，未安装时 fallback 到 `openssl`（需支持 SM2/SM3）；若两者都不可用则脚本失败退出 |
| **节点运行（国密）** | `linux-x86_64` | `bin/easysearch` 可识别并加载对应 JNI 原生库 |
| **节点运行（国密）** | `linux-aarch64` | `bin/easysearch` 可识别并加载对应 JNI 原生库 |
| **节点运行（国密）** | `darwin-x86_64` | `bin/easysearch` 可识别并加载对应 JNI 原生库 |
| **节点运行（国密）** | `darwin-aarch64` | `bin/easysearch` 可识别并加载对应 JNI 原生库 |

> 以上“节点运行”平台为启动脚本识别的目标平台；实际可用性仍取决于发行包中是否包含对应 native 库文件。

---

## 合规参考

| 标准 | 说明 |
|------|------|
| **GM/T 0024-2014** | SSL VPN 技术规范 |
| **GM/T 0028-2014** | 密码模块安全技术要求 |
| **JR/T 0197-2020** | 金融行业密码应用技术标准 |

> 生产环境部署前，建议与合规团队确认具体的国密要求和认证需求。

---

## 延伸阅读

- [TLS 安全配置]({{< relref "./tls-secure.md" >}}) — 标准 TLS 证书管理
- [安全配置]({{< relref "/docs/deployment/config/node-settings/security-settings" >}}) — `easysearch.yml` 安全参数总览
- [安全 API]({{< relref "/docs/operations/security/" >}}) — 用户、角色、权限管理

