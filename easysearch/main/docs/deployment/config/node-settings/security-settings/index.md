---
title: "安全配置"
date: 0001-01-01
summary: "安全配置 #  本页介绍 easysearch.yml 中与安全模块相关的配置项，包括 TLS 加密、审计日志、节点证书、REST API 权限等。这些都是静态设置，修改后需要重启节点生效。
 用户、角色、权限等安全管理操作请参见 安全 API。 详细的 TLS 证书管理请参见 TLS 安全配置。 国密算法配置请参见 国密配置。
  安全模块开关 #  security.enabled #  security.enabled: true    项目 说明     参数 security.enabled   默认值 true   属性 静态   说明 是否启用安全模块。启用后所有 API 请求需要认证，节点间通信需要 TLS 加密     生产环境必须启用安全模块。关闭安全意味着任何人都可以读写集群数据。
  审计日志 #  security.audit.type #  security."
---


# 安全配置

本页介绍 `easysearch.yml` 中与安全模块相关的配置项，包括 TLS 加密、审计日志、节点证书、REST API 权限等。这些都是**静态设置**，修改后需要重启节点生效。

> 用户、角色、权限等安全管理操作请参见 [安全 API]({{< relref "/docs/operations/security/" >}})。
> 详细的 TLS 证书管理请参见 [TLS 安全配置]({{< relref "/docs/deployment/advanced-config/tls-secure.md" >}})。
> 国密算法配置请参见 [国密配置]({{< relref "/docs/deployment/advanced-config/guomi.md" >}})。

---

## 安全模块开关

### security.enabled

```yaml
security.enabled: true
```

| 项目 | 说明 |
|------|------|
| **参数** | `security.enabled` |
| **默认值** | `true` |
| **属性** | 静态 |
| **说明** | 是否启用安全模块。启用后所有 API 请求需要认证，节点间通信需要 TLS 加密 |

> **生产环境必须启用安全模块**。关闭安全意味着任何人都可以读写集群数据。

---

## 审计日志

### security.audit.type

```yaml
security.audit.type: noop
```

| 项目 | 说明 |
|------|------|
| **参数** | `security.audit.type` |
| **默认值** | `noop`（发行包默认配置） |
| **属性** | 静态 |
| **说明** | 审计日志类型。`noop` 表示不记录审计日志，`internal` 表示记录到 Easysearch 索引中 |

| 值 | 行为 |
|------|------|
| `noop` | 不记录审计日志（默认） |
| `internal` | 将审计事件记录到 Easysearch 内部索引 |
| `log4j` | 将审计事件输出到 log4j 日志文件 |
| `webhook` | 将审计事件发送到 Webhook 端点 |
| `debug` | 调试模式，仅在开发环境使用 |

---

## 配置可见性控制

### security.settings.expose

```yaml
security.settings.expose:
  - security.audit.type
  - security.ssl.http.enabled
```

| 项目 | 说明 |
|------|------|
| **参数** | `security.settings.expose` |
| **默认值** | 空 |
| **属性** | 静态 |
| **说明** | 控制哪些 `security.*` 配置项可以在设置相关 API 响应中显示；未显式放开的 `security.*` 配置默认会被过滤 |

`security.settings.expose` 支持以下两类模式：

- 精确配置名，例如 `security.audit.type`
- 通配符模式，例如 `security.ssl.http.*`

上面的示例含义是：

- 暴露 `security.audit.type`
- 暴露 `security.ssl.http.enabled`

> 结合当前实现，`security.settings.expose` 用于声明“允许显示”的配置项或通配符模式，不应在该配置中直接使用 `!pattern` 语法。

> 修改该配置后需要重启节点生效。生产环境中应只暴露排障或观测确有需要的安全配置项，避免使用过宽的通配符模式暴露敏感信息。

---

## Transport 层 TLS

Transport 层用于节点间内部通信。**安全模块启用时，Transport TLS 必须配置**。

| 参数 | 说明 |
|------|------|
| `security.ssl.transport.cert_file` | 节点证书文件（PEM 格式），相对于 `config/` 目录 |
| `security.ssl.transport.key_file` | 节点私钥文件（PEM 格式） |
| `security.ssl.transport.ca_file` | CA 根证书文件，用于验证其他节点的证书 |
| `security.ssl.transport.skip_domain_verify` | 是否跳过证书域名验证。`true` 表示只验证证书签发链，不验证 CN/SAN |

```yaml
security.ssl.transport.cert_file: instance.crt
security.ssl.transport.key_file: instance.key
security.ssl.transport.ca_file: ca.crt
security.ssl.transport.skip_domain_verify: true
```

### 证书要求

- 每个节点都需要有自己的证书和私钥。
- 所有节点的证书必须由**同一个 CA** 签发（或证书链中可追溯到同一根 CA）。
- 证书文件路径相对于 `config/` 目录。

---

## HTTP 层 TLS

HTTP 层用于客户端（REST API）通信。启用后客户端需要使用 `https://` 连接。

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `security.ssl.http.enabled` | `true` | 是否启用 HTTPS（初始化后默认开启） |
| `security.ssl.http.cert_file` | — | HTTP 层证书文件 |
| `security.ssl.http.key_file` | — | HTTP 层私钥文件 |
| `security.ssl.http.ca_file` | — | CA 根证书文件 |
| `security.ssl.http.clientauth_mode` | `OPTIONAL` | 客户端证书认证模式 |

```yaml
security.ssl.http.enabled: true
security.ssl.http.cert_file: instance.crt
security.ssl.http.key_file: instance.key
security.ssl.http.ca_file: ca.crt
security.ssl.http.clientauth_mode: OPTIONAL
```

### clientauth_mode 说明

| 值 | 行为 |
|------|------|
| `NONE` | 不要求客户端证书 |
| `OPTIONAL` | 客户端可以提供证书，但不强制（默认） |
| `REQUIRE` | 强制要求客户端提供证书（双向 TLS / mTLS） |

> 通常使用 `OPTIONAL`，仅在需要 mTLS 的高安全场景使用 `REQUIRE`。

---

## 节点 DN（Distinguished Name）

### security.nodes_dn

```yaml
security.nodes_dn:
  - "CN=node-1.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
  - "CN=node-2.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
  - "CN=node-3.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
```

| 项目 | 说明 |
|------|------|
| **参数** | `security.nodes_dn` |
| **默认值** | 空列表 |
| **属性** | 静态 |
| **说明** | 允许加入集群的节点证书 DN 列表。只有证书 DN 匹配此列表的节点才能加入集群。支持通配符 |

**通配符示例**：

```yaml
security.nodes_dn:
  - "CN=*.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
```

### 查看证书 DN

```bash
openssl x509 -in instance.crt -subject -noout
```

---

## Admin DN

### security.authcz.admin_dn

```yaml
security.authcz.admin_dn:
  - "CN=admin.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
```

| 项目 | 说明 |
|------|------|
| **参数** | `security.authcz.admin_dn` |
| **默认值** | 空列表 |
| **属性** | 静态 |
| **说明** | Admin 客户端证书 DN 列表。持有这些 DN 证书的客户端拥有完全的集群管理权限，可以执行安全配置变更等特权操作 |

> Admin 证书用于运行 `securityadmin.sh` 或其他需要特权访问的管理工具。

---

## REST API 配置

### security.restapi.roles_enabled

```yaml
security.restapi.roles_enabled: ["superuser", "security_rest_api_access"]
```

| 项目 | 说明 |
|------|------|
| **参数** | `security.restapi.roles_enabled` |
| **默认值** | — |
| **属性** | 静态 |
| **说明** | 允许访问安全 REST API（`_security/*`）的角色列表。只有拥有这些角色的用户才能通过 API 管理安全配置 |

---

## 安全索引初始化

### security.allow_default_init_securityindex

```yaml
security.allow_default_init_securityindex: false
```

| 项目 | 说明 |
|------|------|
| **参数** | `security.allow_default_init_securityindex` |
| **默认值** | `false` |
| **属性** | 静态 |
| **说明** | 是否允许在首次启动时自动初始化安全索引（`.security` 系统索引）。设为 `true` 时集群启动时会自动创建安全索引并初始化默认配置 |

---

## 系统索引保护

### security.system_indices

```yaml
security.system_indices.enabled: true
security.system_indices.indices: [".infini-*"]
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `security.system_indices.enabled` | `true` | 是否启用系统索引保护（初始化后默认开启） |
| `security.system_indices.indices` | — | 受保护的系统索引模式列表。匹配的索引只能通过系统内部访问，普通用户无法直接读写 |

---

## security 目录配置文件

除了 `easysearch.yml` 中的安全配置，`config/security/` 目录还包含安全模块的本地配置文件：

| 文件 | 说明 |
|------|------|
| `user.yml` | 内置用户定义（密码哈希由初始化脚本生成） |
| `roles.yml` | 角色定义 |
| `roles_mapping.yml` | 角色映射（用户/后端角色 → 安全角色） |
| `action_groups.yml` | 权限组定义 |
| `tenants.yml` | 租户定义 |
| `config.yml` | 认证与授权后端配置（LDAP、SAML 等） |

> 这些文件在 `bin/initialize.sh` 初始化时自动生成。

---

## 完整配置示例

### 标准生产环境

```yaml
# ---- 安全模块 ----
security.enabled: true
security.audit.type: noop

# ---- Transport TLS ----
security.ssl.transport.cert_file: instance.crt
security.ssl.transport.key_file: instance.key
security.ssl.transport.ca_file: ca.crt
security.ssl.transport.skip_domain_verify: true

# ---- HTTP TLS ----
security.ssl.http.enabled: true
security.ssl.http.cert_file: instance.crt
security.ssl.http.key_file: instance.key
security.ssl.http.ca_file: ca.crt
security.ssl.http.clientauth_mode: OPTIONAL

# ---- 访问控制 ----
security.allow_default_init_securityindex: true
security.nodes_dn:
  - "CN=*.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
security.authcz.admin_dn:
  - "CN=admin.infini.cloud,OU=UNIT,O=ORG,L=NI,ST=FI,C=IN"
security.restapi.roles_enabled: ["superuser", "security_rest_api_access"]

# ---- 系统索引保护 ----
security.system_indices.enabled: true
security.system_indices.indices: [".infini-*"]
```

---

## 延伸阅读

- [TLS 安全配置]({{< relref "/docs/deployment/advanced-config/tls-secure.md" >}}) — 详细的证书生成与管理
- [国密配置]({{< relref "/docs/deployment/advanced-config/guomi.md" >}}) — SM2/SM3/SM4 国密 TLS
- [安全 API]({{< relref "/docs/operations/security/" >}}) — 用户、角色、权限管理

