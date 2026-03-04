---
title: "国密与国产化"
date: 0001-01-01
description: "SM2/SM3/SM4 国密算法全量支持，深度适配国产 CPU 与操作系统，满足信创与等保合规。"
summary: "国密与国产化 #  Easysearch 围绕国产密码算法与国产化生态深度设计，为政企与关键行业提供可落地、可审计、可替代的搜索基础能力。
 核心能力 #     能力 说明     国密算法 全量支持 SM2/SM3/SM4，替代 RSA/AES/SHA   国产 CPU 适配鲲鹏、飞腾、海光、龙芯、兆芯   国产 OS 适配银河麒麟、统信 UOS、中标麒麟   等保合规 满足等保三级及信创合规要求     国密算法 #  Easysearch 基于 铜锁（Tongsuo）实现完整的国密 TLS 能力。
SM2 — 非对称加密 #  替代 RSA/ECDSA，用于：
 TLS 证书签名与验证 节点间身份认证 客户端证书认证  SM3 — 哈希算法 #  替代 SHA-256，用于：
 密码存储与验证 数据完整性校验 审计日志防篡改  SM4 — 对称加密 #  替代 AES，用于："
---


# 国密与国产化

Easysearch 围绕国产密码算法与国产化生态深度设计，为政企与关键行业提供可落地、可审计、可替代的搜索基础能力。

---

## 核心能力

| 能力 | 说明 |
|------|------|
| **国密算法** | 全量支持 SM2/SM3/SM4，替代 RSA/AES/SHA |
| **国产 CPU** | 适配鲲鹏、飞腾、海光、龙芯、兆芯 |
| **国产 OS** | 适配银河麒麟、统信 UOS、中标麒麟 |
| **等保合规** | 满足等保三级及信创合规要求 |

---

## 国密算法

Easysearch 基于[铜锁（Tongsuo）](https://release.infinilabs.com/easysearch/archive/tongsuo/)实现完整的国密 TLS 能力。

### SM2 — 非对称加密

替代 RSA/ECDSA，用于：
- TLS 证书签名与验证
- 节点间身份认证
- 客户端证书认证

### SM3 — 哈希算法

替代 SHA-256，用于：
- 密码存储与验证
- 数据完整性校验
- 审计日志防篡改

### SM4 — 对称加密

替代 AES，用于：
- 节点间通信加密（Transport 层）
- HTTPS 传输加密（HTTP 层）
- 敏感数据存储

### 加密套件

| 套件 | 说明 |
|------|------|
| `TLS_SM4_GCM_SM3` | SM4-GCM 加密 + SM3 哈希（推荐） |
| `TLS_SM4_CCM_SM3` | SM4-CCM 加密 + SM3 哈希 |

---

## 国产化适配

### 国产处理器

| 处理器 | 架构 | 状态 |
|--------|------|------|
| **鲲鹏** (Kunpeng) | ARM64 | ✅ 认证通过 |
| **飞腾** (Phytium) | ARM64 | ✅ 认证通过 |
| **海光** (Hygon) | x86 | ✅ 认证通过 |
| **龙芯** (Loongson) | LoongArch | ✅ 认证通过 |
| **兆芯** (Zhaoxin) | x86 | ✅ 认证通过 |

### 国产操作系统

| 操作系统 | 状态 |
|----------|------|
| **银河麒麟** (Kylin OS) | ✅ 互认证 |
| **统信 UOS** (UnionTech OS) | ✅ 互认证 |
| **中标麒麟** (NeoKylin) | ✅ 互认证 |

### 生态兼容

可与国产数据库、中间件、安全产品协同部署，融入整体国产化解决方案。

---

## 合规能力

| 维度 | 能力 |
|------|------|
| **传输加密** | SM4 对称加密 + SM2 身份认证 |
| **密码存储** | SM3 哈希，不可逆 |
| **访问控制** | RBAC 角色权限，支持文档/字段级 |
| **审计日志** | 完整操作记录，SM3 防篡改 |
| **数据完整性** | SM3 校验 |

### 合规标准

| 标准 | 说明 |
|------|------|
| GM/T 0024-2014 | SSL VPN 技术规范 |
| GM/T 0028-2014 | 密码模块安全技术要求 |
| JR/T 0197-2020 | 金融行业密码应用技术标准 |

---

## 应用场景

### 政务与公共服务

满足政务系统对国密算法与国产化环境的强制要求，支撑电子政务、公共数据开放平台。

### 金融与关键基础设施

金融、能源、电信等行业，构建符合监管要求的搜索与分析能力。

### 信创替代

在国产化环境中搭建搜索平台，平滑迁移既有 Elasticsearch 应用。

### 行业信息化

作为国产化体系中的搜索引擎组件，支撑数据中台、日志分析、全文检索等场景。

---

## 快速开始

### 0. Linux 宿主机安装 Tongsuo（推荐）

```bash
chmod +x bin/install-tongsuo-local.sh
bin/install-tongsuo-local.sh --version 8.4.0
source ~/.tongsuo-switch.sh
use_tongsuo
```

> `install-tongsuo-local.sh` 仅支持 Linux 非 root 用户，默认安装到 `$HOME/local/tongsuo`，不覆盖系统 OpenSSL。完整说明见[国密配置指南]({{< relref "../deployment/advanced-config/guomi" >}})。
>
> 可选：`bin/install-tongsuo-local.sh --archive-file /path/to/tongsuo-8.4.0.tar.gz`（离线安装）。

### 1. 生成国密证书

```bash
bin/generate-tlcp-certs.sh /path/to/output "changeit"
```

### 2. 配置国密 TLS

```yaml
security.enabled: true
security.ssl.use_tongsuo: true

# HTTP 层
security.ssl.http.enabled: true
security.ssl.http.tlcp.sign_certificate: server_sign.crt
security.ssl.http.tlcp.sign_key: server_sign.key
security.ssl.http.tlcp.enc_certificate: server_enc.crt
security.ssl.http.tlcp.enc_key: server_enc.key
security.ssl.http.tlcp.trusted_ca_certificate: ca_chain.crt

# Transport 层
security.ssl.transport.enabled: true
security.ssl.transport.tlcp.sign_certificate: server_sign.crt
security.ssl.transport.tlcp.sign_key: server_sign.key
security.ssl.transport.tlcp.enc_certificate: server_enc.crt
security.ssl.transport.tlcp.enc_key: server_enc.key
security.ssl.transport.tlcp.trusted_ca_certificate: ca_chain.crt
```

### 3. 一键初始化（单节点 TLCP）

如需在单节点场景快速完成证书生成、TLCP 配置写入和初始密码设置，可直接使用：

```bash
cd /path/to/easysearch
EASYSEARCH_INITIAL_ADMIN_PASSWORD='Admin@123Test!' bin/initialize.sh --tlcp -s
```

> 适用于首次初始化（`config/` 下没有旧的 `.crt/.key` 文件）。
>
> `initialize.sh --tlcp` 不会自动安装 Tongsuo。Linux 上请先确保 `tongsuo` 可用；macOS 可使用具备 SM2/SM3 能力的 `openssl` 或已安装的 `tongsuo`。
>
> 若是多节点部署，请使用 `bin/initialize-cluster.sh --tlcp`。

### 4. 验证配置

```bash
bin/tlcp-curl.sh https://localhost:9200/_security/sslinfo?pretty \
  -u admin:changeit
```

> `tlcp-curl.sh` 支持 `--url <url>` 或位置参数 `<url>`（二选一）。
>
> 仅当服务端配置 `security.ssl.http.clientauth_mode: REQUIRE` 时，客户端证书访问才是必需的；默认场景可仅使用用户名密码。
>
> 如需显式双证书访问：
>
> ```bash
> bin/tlcp-curl.sh https://localhost:9200/_security/sslinfo?pretty \
>   -u admin:changeit \
>   --cert config/client_sign.crt --key config/client_sign.key \
>   --enc-cert config/client_enc.crt --enc-key config/client_enc.key \
>   --ca config/ca_chain.crt
> ```

确认 `ssl_provider` 为 `Tongsuo_Security_Provider`。

### 5. 重置 admin 密码（TLCP 模式）

当 `security.ssl.use_tongsuo: true` 时，`bin/reset_admin_password.sh` 会自动使用 `bin/tlcp-curl.sh` 与证书调用安全 API：

```bash
bin/reset_admin_password.sh
# 或指定地址
bin/reset_admin_password.sh 'https://127.0.0.1:9200'
```

> 完整配置指南见 [国密配置]({{< relref "../deployment/advanced-config/guomi" >}})。

---

## 相关文档

- [国密配置指南]({{< relref "../deployment/advanced-config/guomi" >}})：完整的 TLCP 双证书配置
- [TLS 安全配置]({{< relref "../deployment/advanced-config/tls-secure" >}})：标准 TLS 配置
- [安全模块]({{< relref "../operations/security" >}})：认证授权与访问控制

