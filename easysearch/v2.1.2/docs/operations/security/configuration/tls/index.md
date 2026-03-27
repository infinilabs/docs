---
title: "证书配置"
date: 0001-01-01
summary: "配置 TLS 证书 #  Easysearch 可以通过启用 TLS 传输加密来保护您数据的网络传输安全。 TLS 的相关设置在配置文件 easysearch.yml 中进行，主要包括两个部分：传输层和 HTTP 层。
 传输层（Transport）：节点之间的通信。传输层 TLS 是必需的（默认启用）。 HTTP 层：客户端与节点之间的通信。HTTP 层 TLS 是可选的（默认关闭）。  默认的配置如下：
security.ssl.transport.cert_file: instance.crt security.ssl.transport.key_file: instance.key security.ssl.transport.ca_file: ca.crt security.ssl.http.enabled: true security.ssl.http.cert_file: instance.crt security.ssl.http.key_file: instance.key security.ssl.http.ca_file: ca.crt 一键生成证书 #  启用 TLS 需要设置证书才能工作，通过执行命令 ./bin/initialize.sh 可以一键生成 TLS 证书，如下：
➨ ./bin/initialize.sh Generating RSA private key, 2048 bit long modulus .......................+++ ...............................+++ e is 65537 (0x10001) Generating RSA private key, 2048 bit long modulus ."
---


# 配置 TLS 证书

Easysearch 可以通过启用 TLS 传输加密来保护您数据的网络传输安全。
TLS 的相关设置在配置文件 `easysearch.yml` 中进行，主要包括两个部分：传输层和 HTTP 层。

- **传输层（Transport）**：节点之间的通信。传输层 TLS 是必需的（默认启用）。
- **HTTP 层**：客户端与节点之间的通信。HTTP 层 TLS 是可选的（默认关闭）。

默认的配置如下：

```yaml
security.ssl.transport.cert_file: instance.crt
security.ssl.transport.key_file: instance.key
security.ssl.transport.ca_file: ca.crt
security.ssl.http.enabled: true
security.ssl.http.cert_file: instance.crt
security.ssl.http.key_file: instance.key
security.ssl.http.ca_file: ca.crt
```

## 一键生成证书

启用 TLS 需要设置证书才能工作，通过执行命令 `./bin/initialize.sh` 可以一键生成 TLS 证书，如下：

```
➨  ./bin/initialize.sh
Generating RSA private key, 2048 bit long modulus
.......................+++
...............................+++
e is 65537 (0x10001)
Generating RSA private key, 2048 bit long modulus
.......................................................................................................................................................................................+++
......................................................+++
e is 65537 (0x10001)
Signature ok
subject=/C=IN/ST=FI/L=NI/O=ORG/OU=UNIT/CN=infini.cloud
Getting CA Private Key
                DNS:infini.cloud, DNS:*.infini.cloud
➨  ls config
ca.crt                 easysearch.yml         instance.key           log4j2.properties
ca.key                 easysearch.yml.example jvm.options            security
easysearch.keystore    instance.crt           jvm.options.d
```

## TLS 启用开关

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.transport.enabled` | 是否启用传输层 TLS。 | `true` |
| `security.ssl.http.enabled` | 是否启用 HTTP 层 TLS。 | `false` |

## X.509 PEM 证书和 PKCS\#8 密钥

下面是 Easysearch 使用 PEM 格式证书和 PKCS#8 私钥的详细参数说明。所有证书和密钥文件必须位于 `config` 目录下，使用相对路径指定。

### 传输层 TLS（PEM）

| 名称 | 描述 |
| :--- | :--- |
| `security.ssl.transport.key_file` | 节点私钥文件（PKCS#8，PEM 格式）的路径。必填。 |
| `security.ssl.transport.key_secret` | 私钥文件的密码。如果密钥没有密码保护，可忽略。选填。 |
| `security.ssl.transport.cert_file` | 节点证书文件（PEM 格式）的路径。必填。 |
| `security.ssl.transport.ca_file` | 根证书（CA）文件（PEM 格式）的路径。必填。 |

### HTTP 层 TLS（PEM）

| 名称 | 描述 |
| :--- | :--- |
| `security.ssl.http.key_file` | 节点私钥文件（PKCS#8，PEM 格式）的路径。必填。 |
| `security.ssl.http.key_secret` | 私钥文件的密码。如果密钥没有密码保护，可忽略。选填。 |
| `security.ssl.http.cert_file` | 节点证书文件（PEM 格式）的路径。必填。 |
| `security.ssl.http.ca_file` | 根证书（CA）文件（PEM 格式）的路径。必填。 |

## JKS/PKCS12 Keystore 和 Truststore

除了 PEM 格式，Easysearch 还支持使用 JKS 或 PKCS12 格式的 Keystore 和 Truststore 来存储证书和密钥。所有 Keystore/Truststore 文件必须位于 `config` 目录下，使用相对路径指定。

> **注意：** PEM 格式和 Keystore 格式是互斥的，每一层只能选择其中一种方式配置证书。

### 传输层 TLS（Keystore）

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.transport.keystore_filepath` | Keystore 文件的路径。必填（若使用 Keystore 方式）。 | - |
| `security.ssl.transport.keystore_password` | Keystore 的密码。 | `changeit` |
| `security.ssl.transport.keystore_keypassword` | Keystore 中私钥的密码（如果与 Keystore 密码不同）。 | 与 `keystore_password` 相同 |
| `security.ssl.transport.keystore_alias` | Keystore 中要使用的证书别名。如果 Keystore 包含多个证书条目，可指定别名。 | 使用第一个条目 |
| `security.ssl.transport.keystore_type` | Keystore 的类型。 | `JKS` |
| `security.ssl.transport.truststore_filepath` | Truststore 文件的路径。必填（若使用 Keystore 方式）。 | - |
| `security.ssl.transport.truststore_password` | Truststore 的密码。 | `changeit` |
| `security.ssl.transport.truststore_alias` | Truststore 中要使用的 CA 证书别名。 | 使用第一个条目 |
| `security.ssl.transport.truststore_type` | Truststore 的类型。 | `JKS` |

### HTTP 层 TLS（Keystore）

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.http.keystore_filepath` | Keystore 文件的路径。必填（若使用 Keystore 方式）。 | - |
| `security.ssl.http.keystore_password` | Keystore 的密码。 | `changeit` |
| `security.ssl.http.keystore_keypassword` | Keystore 中私钥的密码（如果与 Keystore 密码不同）。 | 与 `keystore_password` 相同 |
| `security.ssl.http.keystore_alias` | Keystore 中要使用的证书别名。 | 使用第一个条目 |
| `security.ssl.http.keystore_type` | Keystore 的类型。 | `JKS` |
| `security.ssl.http.truststore_filepath` | Truststore 文件的路径。当 `clientauth_mode` 为 `REQUIRE` 时必填。 | - |
| `security.ssl.http.truststore_password` | Truststore 的密码。 | `changeit` |
| `security.ssl.http.truststore_alias` | Truststore 中要使用的 CA 证书别名。 | 使用第一个条目 |
| `security.ssl.http.truststore_type` | Truststore 的类型。 | `JKS` |

### Keystore 配置示例

```yaml
# 传输层使用 JKS Keystore
security.ssl.transport.keystore_filepath: node-keystore.jks
security.ssl.transport.keystore_password: mypassword
security.ssl.transport.truststore_filepath: node-truststore.jks
security.ssl.transport.truststore_password: mypassword

# HTTP 层使用 PKCS12 Keystore
security.ssl.http.enabled: true
security.ssl.http.keystore_filepath: node-keystore.p12
security.ssl.http.keystore_password: mypassword
security.ssl.http.keystore_type: PKCS12
security.ssl.http.truststore_filepath: node-truststore.p12
security.ssl.http.truststore_password: mypassword
security.ssl.http.truststore_type: PKCS12
```

## 扩展密钥用途（Extended Key Usage）

当启用扩展密钥用途时，传输层可以为服务端和客户端使用不同的证书。这在需要严格区分节点间入站和出站连接身份的场景中非常有用。

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.transport.extended_key_usage_enabled` | 是否启用扩展密钥用途，启用后传输层将分别使用独立的服务端和客户端证书。 | `false` |

### 服务端/客户端独立 PEM 证书

当启用扩展密钥用途时，可以为传输层的服务端和客户端分别指定 PEM 证书文件：

| 名称 | 描述 |
| :--- | :--- |
| `security.ssl.transport.server.key_file` | 传输层服务端私钥文件路径。 |
| `security.ssl.transport.server.key_secret` | 服务端私钥密码。选填。 |
| `security.ssl.transport.server.cert_file` | 传输层服务端证书文件路径。 |
| `security.ssl.transport.server.ca_file` | 服务端 CA 证书文件路径。 |
| `security.ssl.transport.client.key_file` | 传输层客户端私钥文件路径。 |
| `security.ssl.transport.client.key_secret` | 客户端私钥密码。选填。 |
| `security.ssl.transport.client.cert_file` | 传输层客户端证书文件路径。 |
| `security.ssl.transport.client.ca_file` | 客户端 CA 证书文件路径。 |

### 服务端/客户端独立 Keystore 别名

当启用扩展密钥用途时，也可以通过 Keystore 别名来区分服务端和客户端证书。此时需要同时指定所有四个别名：

| 名称 | 描述 |
| :--- | :--- |
| `security.ssl.transport.server.keystore_alias` | Keystore 中服务端证书的别名。必填。 |
| `security.ssl.transport.server.keystore_keypassword` | 服务端私钥的密码。默认使用 `keystore_password`。 |
| `security.ssl.transport.client.keystore_alias` | Keystore 中客户端证书的别名。必填。 |
| `security.ssl.transport.client.keystore_keypassword` | 客户端私钥的密码。默认使用 `keystore_password`。 |
| `security.ssl.transport.server.truststore_alias` | Truststore 中服务端 CA 证书的别名。必填。 |
| `security.ssl.transport.client.truststore_alias` | Truststore 中客户端 CA 证书的别名。必填。 |

### 扩展密钥用途配置示例

```yaml
# 使用 PEM 格式的独立服务端/客户端证书
security.ssl.transport.extended_key_usage_enabled: true
security.ssl.transport.server.key_file: transport-server.key
security.ssl.transport.server.cert_file: transport-server.crt
security.ssl.transport.server.ca_file: ca.crt
security.ssl.transport.client.key_file: transport-client.key
security.ssl.transport.client.cert_file: transport-client.crt
security.ssl.transport.client.ca_file: ca.crt
```

## 配置节点证书

安全模块需要识别集群节点间的请求（即节点之间的通信）。配置节点证书的最简单方法是在 `easysearch.yml` 中列出这些证书的可分辨名称（DN）。所有 DN 都必须包含在所有节点上的 `easysearch.yml` 中。安全模块支持通配符和正则表达式：

```yaml
security.nodes_dn:
  - "CN=node.other.com,OU=SSL,O=Test,L=Test,C=DE"
  - "CN=*.example.com,OU=SSL,O=Test,L=Test,C=DE"
  - "CN=elk-devcluster*"
  - "/CN=.*regex/"
```

如果您的节点证书在 SAN 部分中具有 OID 标识符，则可以省略此配置。

相关设置：

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.nodes_dn_dynamic_config_enabled` | 是否允许通过 REST API 动态更新 `nodes_dn` 配置。 | `false` |
| `security.cert.oid` | 自定义 OID 用于在节点证书中标识节点身份。如果节点证书的 SAN 中包含此 OID 的值，则无需配置 `security.nodes_dn`。 | `1.2.3.4.5.5` |

## 配置管理证书

管理员证书是具有执行管理任务的提升权限的常规客户端证书。您需要管理员证书才能使用 HTTP API 更改安全配置。管理员证书在 `easysearch.yml` 中通过声明其 DN 进行配置：

```yaml
security.authcz.admin_dn:
  - CN=admin,OU=SSL,O=Test,L=Test,C=DE
```

出于安全原因，您不能在此处使用通配符或正则表达式。

## 证书身份模拟（Impersonation）

安全模块支持基于证书 DN 的身份模拟功能，允许经过认证的用户或传输请求以另一个用户的身份执行操作。

| 名称 | 描述 |
| :--- | :--- |
| `security.authcz.impersonation_dn` | 映射传输层证书 DN 到允许模拟的用户列表。 |
| `security.authcz.rest_impersonation_user` | 映射 REST 层用户到允许模拟的用户列表。 |

### 模拟配置示例

```yaml
security.authcz.impersonation_dn:
  "CN=spock,OU=client,O=client,L=Test,C=DE":
    - "worf"
    - "naomi"

security.authcz.rest_impersonation_user:
  "admin":
    - "monitor_user"
    - "report_user"
```

## 主机名验证和 DNS 查找

除了根据根 CA 和/或中间 CA 验证 TLS 证书外，安全插件还可以对传输层应用其他检查。

禁用 `skip_domain_verify` 后，安全模块将验证通信伙伴的主机名是否与证书中的主机名匹配。

主机名取自证书的 `subject` 或 `SAN` 条目。例如，如果节点的主机名为 `node-0.example.com`，则 TLS 证书中的主机名也必须设置为 `node-0.example.com`。否则，将引发错误：

```
[ERROR][c.a.o.s.s.t.SecuritySSLNettyTransport] [WX6omJY] SSL Problem No name matching <hostname> found
[ERROR][c.a.o.s.s.t.SecuritySSLNettyTransport] [WX6omJY] SSL Problem Received fatal alert: certificate_unknown
```

此外，启用 `resolve_hostname` 后，安全模块会根据您的 DNS 解析（经过验证的）主机名。如果主机名未解析，则会引发错误。

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.transport.skip_domain_verify` | 是否跳过验证传输层上的主机名。 | `true` |
| `security.ssl.transport.resolve_hostname` | 是否在传输层上根据 DNS 解析主机名。仅在同时启用主机名验证时（`skip_domain_verify` 为 `false`）才有效。 | `true` |

## 客户端认证

启用 TLS 客户端身份验证后，HTTP 客户端可以将 TLS 证书与 HTTP 请求一起发送，以向安全模块提供身份信息。TLS 客户端认证主要有以下使用场景：

- 在使用 HTTP 管理 API 时提供管理员证书。
- 根据客户端证书配置角色和权限。
- 为 INFINI Console、Logstash 或 Beats 等工具提供身份信息。

TLS 客户端认证有三种模式：

| 模式 | 描述 |
| :--- | :--- |
| `NONE` | 安全模块不接受 TLS 客户端证书。如果发送了一个，则将其丢弃。 |
| `OPTIONAL` | 安全模块接受 TLS 客户端证书（如果已发送），但不强制要求。**默认值。** |
| `REQUIRE` | 安全模块仅在发送有效的客户端 TLS 证书时接受 HTTP 请求。 |

对于 HTTP 管理 API，客户端身份验证模式至少必须是 `OPTIONAL`。

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.http.clientauth_mode` | 要使用的 TLS 客户端身份验证模式。可选值：`NONE`、`OPTIONAL`、`REQUIRE`。 | `OPTIONAL` |

### 样例

```bash
curl -k --cert config/instance.crt --key config/instance.key \\
  -XDELETE 'https://localhost:9200/.infini-*/' -u admin:xxxxxxxxxxxx
```

## 证书吊销列表（CRL）

Easysearch 支持通过 CRL（证书吊销列表）和 OCSP（在线证书状态协议）来验证 HTTP 层客户端证书的有效性。

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.http.crl.file_path` | CRL 文件的路径（PEM 格式），文件必须位于 `config` 目录下。 | - |
| `security.ssl.http.crl.validate` | 是否启用 CRL 验证。 | `false` |
| `security.ssl.http.crl.prefer_crlfile_over_ocsp` | 优先使用本地 CRL 文件而非 OCSP 进行证书吊销检查。 | `false` |
| `security.ssl.http.crl.check_only_end_entities` | 仅验证终端实体证书，不验证 CA 证书链中的中间证书。 | `true` |
| `security.ssl.http.crl.disable_ocsp` | 禁用 OCSP 检查。 | `false` |
| `security.ssl.http.crl.disable_crldp` | 禁用证书中 CRL 分发点（CRLDP）的自动下载。 | `false` |
| `security.ssl.http.crl.validation_date` | 用于验证的日期（主要用于测试目的）。格式为 ISO 8601 日期字符串。 | 当前日期 |

## 加密算法和协议

您可以限制 HTTP 层和传输层允许的加密算法和 TLS 协议。例如，您可以只允许强密码算法，并将 TLS 版本限制为最新版本。

如果未配置此设置，密码和 TLS 版本将在通信双方之间自动协商，这在某些情况下可能会导致使用较弱的密码套件。

| 名称 | 描述 |
| :--- | :--- |
| `security.ssl.http.enabled_ciphers` | 数组，HTTP 层启用的 TLS 密码套件。仅支持 Java 格式。 |
| `security.ssl.http.enabled_protocols` | 数组，HTTP 层启用的 TLS 协议。仅支持 Java 格式。 |
| `security.ssl.transport.enabled_ciphers` | 数组，传输层启用的 TLS 密码套件。仅支持 Java 格式。 |
| `security.ssl.transport.enabled_protocols` | 数组，传输层启用的 TLS 协议。仅支持 Java 格式。 |

默认启用的协议为 `TLSv1.3`、`TLSv1.2` 和 `TLSv1.1`。不安全的 `TLSv1` 协议默认禁用。

### 样例

```yaml
security.ssl.http.enabled_ciphers:
  - "TLS_AES_128_GCM_SHA256"
  - "TLS_AES_256_GCM_SHA384"
  - "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA"
  - "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256"
  - "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
  - "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA"
  - "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384"
  - "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384"
security.ssl.http.enabled_protocols:
  - "TLSv1.2"
  - "TLSv1.3"
```

如果您需要使用 `TLSv1` 并接受潜在的安全风险，您仍然可以手动启用它：

```yaml
security.ssl.http.enabled_protocols:
  - "TLSv1"
  - "TLSv1.1"
  - "TLSv1.2"
```

## 高级设置

### OpenSSL 支持

Easysearch 支持使用 OpenSSL 作为 TLS 引擎来替代默认的 JDK SSL 实现。OpenSSL 通常可以提供更好的性能。

> **注意：** OpenSSL 支持需要 Java 11 或更低版本，且必须设置系统属性 `es.unsafe.use_netty_default_allocator=true`。Java 12 及以上版本不支持 OpenSSL，将自动使用 JDK SSL。

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.http.enable_openssl_if_available` | 如果 OpenSSL 可用，HTTP 层是否使用 OpenSSL。 | `true` |
| `security.ssl.transport.enable_openssl_if_available` | 如果 OpenSSL 可用，传输层是否使用 OpenSSL。 | `true` |

### 证书热加载

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.cert_reload.enabled` | 是否启用 TLS 证书的热加载。启用后，当证书文件发生变化时，节点将自动加载新证书而无需重启。 | `false` |

### TLS 客户端重协商

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.allow_client_initiated_renegotiation` | 是否允许客户端发起的 TLS 重协商。允许此设置可能会导致 DoS 攻击风险，建议保持默认禁用。 | `false` |

### 传输层主体提取器

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.transport.principal_extractor_class` | 自定义主体提取器类的完全限定类名。用于自定义如何从传输层 TLS 证书中提取用户身份信息。 | - |

### SSL-Only 模式

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl_only` | 仅启用 SSL/TLS 加密，不加载安全插件的其余功能（认证、授权等）。适用于仅需要传输加密而不需要安全控制的场景。 | `false` |

### 双模式（Dual Mode）

| 名称 | 描述 | 默认值 |
| :--- | :--- | :----- |
| `security.ssl.dual_mode.enabled` | 启用 SSL 双模式。当启用时，节点同时接受 SSL 和非 SSL 的传输层连接，便于集群从非加密向加密状态的平滑迁移。 | `false` |

## 完整配置示例

以下是一个使用 PEM 格式证书的完整配置示例：

```yaml
# ===== 传输层 TLS =====
security.ssl.transport.enabled: true
security.ssl.transport.key_file: instance.key
security.ssl.transport.cert_file: instance.crt
security.ssl.transport.ca_file: ca.crt
security.ssl.transport.skip_domain_verify: true

# ===== HTTP 层 TLS =====
security.ssl.http.enabled: true
security.ssl.http.key_file: instance.key
security.ssl.http.cert_file: instance.crt
security.ssl.http.ca_file: ca.crt
security.ssl.http.clientauth_mode: OPTIONAL

# ===== 节点和管理员 DN =====
security.nodes_dn:
  - "CN=*.example.com,OU=SSL,O=Test,L=Test,C=DE"
security.authcz.admin_dn:
  - "CN=admin,OU=SSL,O=Test,L=Test,C=DE"

# ===== 加密算法限制 =====
security.ssl.http.enabled_protocols:
  - "TLSv1.2"
  - "TLSv1.3"
security.ssl.transport.enabled_protocols:
  - "TLSv1.2"
  - "TLSv1.3"

# ===== 证书热加载 =====
security.ssl.cert_reload.enabled: true
```

