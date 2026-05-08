---
title: "TLS 安全配置"
date: 0001-01-01
summary: "TLS 安全配置指南 #  Easysearch 默认启用安全功能，包括 TLS 加密。本文介绍如何配置和管理 TLS 证书，以保障集群通信安全。
TLS 的两层保护 #     层级 配置项 保护范围     HTTP 层 security.ssl.http.* 客户端 ↔ Easysearch REST API   Transport 层 security.ssl.transport.* Easysearch 节点 ↔ 节点     生产环境两层都必须启用。
 使用自签名证书（快速启动） #  bin/initialize.sh -s 会自动生成自签名证书，适合测试和快速验证：
bin/initialize.sh -s # 生成的证书位于 config/ 目录 使用企业 CA 证书（生产推荐） #  1. 准备证书文件 #  需要以下文件：
   文件 说明     ca."
---


# TLS 安全配置指南

Easysearch 默认启用安全功能，包括 TLS 加密。本文介绍如何配置和管理 TLS 证书，以保障集群通信安全。

## TLS 的两层保护

| 层级 | 配置项 | 保护范围 |
|------|--------|----------|
| **HTTP 层** | `security.ssl.http.*` | 客户端 ↔ Easysearch REST API |
| **Transport 层** | `security.ssl.transport.*` | Easysearch 节点 ↔ 节点 |

> 生产环境**两层都必须启用**。

## 使用自签名证书（快速启动）

`bin/initialize.sh -s` 会自动生成自签名证书，适合测试和快速验证：

```bash
bin/initialize.sh -s
# 生成的证书位于 config/ 目录
```

## 使用企业 CA 证书（生产推荐）

### 1. 准备证书文件

需要以下文件：

| 文件 | 说明 |
|------|------|
| `ca.crt` | CA 根证书 |
| `node.crt` | 节点证书 |
| `node.key` | 节点私钥 |
| `admin.crt` | Admin 客户端证书 |
| `admin.key` | Admin 客户端私钥 |

### 2. 配置 easysearch.yml

```yaml
# ========== 启用安全模块 ==========
security.enabled: true
security.audit.type: noop

# ========== Transport 层 TLS ==========
security.ssl.transport.cert_file: node.crt
security.ssl.transport.key_file: node.key
security.ssl.transport.ca_file: ca.crt
security.ssl.transport.skip_domain_verify: true

# ========== HTTP 层 TLS ==========
security.ssl.http.enabled: true
security.ssl.http.cert_file: node.crt
security.ssl.http.key_file: node.key
security.ssl.http.ca_file: ca.crt
security.ssl.http.clientauth_mode: OPTIONAL

# ========== Admin 证书 DN ==========
security.authcz.admin_dn:
  - "CN=admin,OU=DevOps,O=MyCompany,L=Shanghai,ST=Shanghai,C=CN"

security.nodes_dn:
  - "CN=node-*,OU=Infra,O=MyCompany,L=Shanghai,ST=Shanghai,C=CN"

# ========== 其他安全配置 ==========
security.allow_default_init_securityindex: true
security.restapi.roles_enabled: ["superuser", "security_rest_api_access"]
security.system_indices.enabled: true
security.system_indices.indices: [".infini-*"]
```

### 3. 使用 OpenSSL 生成证书

```bash
# 生成 CA
openssl genrsa -out root-ca-key.pem 2048
openssl req -new -x509 -sha256 -key root-ca-key.pem -out root-ca.pem -days 3650 \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/O=MyCompany/OU=Infra/CN=RootCA"

# 生成节点证书
openssl genrsa -out node-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in node-key-temp.pem -topk8 -nocrypt -out node-key.pem
openssl req -new -key node-key.pem -out node.csr \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/O=MyCompany/OU=Infra/CN=node-1"
openssl x509 -req -in node.csr -CA root-ca.pem -CAkey root-ca-key.pem \
  -CAcreateserial -out node.pem -days 730 -sha256

# 生成 admin 证书
openssl genrsa -out admin-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -out admin-key.pem
openssl req -new -key admin-key.pem -out admin.csr \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/O=MyCompany/OU=DevOps/CN=admin"
openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem \
  -CAcreateserial -out admin.pem -days 730 -sha256
```

## TLS 版本与密码套件

### 限制 TLS 版本

```yaml
# 仅允许 TLS 1.2 和 1.3（禁用 1.0/1.1）
security.ssl.http.enabled_protocols:
  - "TLSv1.2"
  - "TLSv1.3"

security.ssl.transport.enabled_protocols:
  - "TLSv1.2"
  - "TLSv1.3"
```

### 限制密码套件

```yaml
security.ssl.http.enabled_ciphers:
  - "TLS_AES_128_GCM_SHA256"
  - "TLS_AES_256_GCM_SHA384"
  - "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
  - "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"
  - "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA"
  - "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA"

security.ssl.transport.enabled_ciphers:
  - "TLS_AES_128_GCM_SHA256"
  - "TLS_AES_256_GCM_SHA384"
  - "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
  - "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"
  - "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA"
  - "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA"
```

## 证书验证

```bash
# 验证证书内容
openssl x509 -in node.pem -text -noout

# 验证证书链
openssl verify -CAfile root-ca.pem node.pem

# 测试 HTTPS 连接
curl -v --cacert root-ca.pem https://localhost:9200
```

## 证书续期

证书到期前需要续期，步骤：

1. 使用同一 CA 签发新证书
2. 替换各节点的 `node.crt` 和 `node.key`
3. 逐节点滚动重启（先重启非 master 节点）

> 建议在监控中设置证书过期告警（提前 30 天）。

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `SSLHandshakeException` | 证书不受信任 | 检查 `ca_file` 是否包含正确的 CA |
| 节点无法加入集群 | Transport 证书 DN 不匹配 | 检查 `nodes_dn` 配置 |
| curl 报证书错误 | 客户端不信任 CA | 使用 `-k` 跳过或 `--cacert` 指定 CA |

## 延伸阅读

- [国密配置]({{< relref "./guomi.md" >}})
- [安全模块总览]({{< relref "/docs/operations/security/" >}})
- [YAML 安全配置]({{< relref "/docs/operations/security/configuration/yaml.md" >}})

