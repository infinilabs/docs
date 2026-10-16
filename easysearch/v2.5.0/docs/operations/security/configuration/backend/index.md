---
title: "后端配置"
date: 0001-01-01
summary: "后端配置 #  配置安全模块的第一步是确定如何验证用户身份。尽管 Easysearch 本身可以充当一个内部用户数据库，但许多人更喜欢集成企业现有的身份认证体系，例如 LDAP/Active Directory 服务器，或两者组合。设置身份验证和授权服务端的主要配置文件位于 config/security/config.yml。它定义了安全模块如何检索用户凭据、如何验证这些凭据以及如何从后端系统获取其他角色（可选）。
config.yml 主要包含三大部分：
config: dynamic: http: ... authc: ... authz: ... HTTP #  http 部分具有以下配置项：
http: anonymous_auth_enabled: &lt;true|false&gt; xff: enabled: &lt;true|false&gt; internalProxies: &#39;&lt;regex pattern&gt;&#39; remoteIpHeader: &#39;&lt;header name&gt;&#39; 匿名认证 #  设置 anonymous_auth_enabled 可以选择是否开启匿名访问。如果启用，当请求中未提供任何凭据时，用户将以 anonymous 身份通过认证，并被分配 anonymous_backendrole 角色。如果禁用匿名身份验证，则至少在 authc 里面提供一个认证后端，否则安全模块将不予初始化。默认为 false。
 提示： 如果启用匿名认证，所有 HTTP 认证器将不会发起 challenge（即不会返回 401 要求客户端提供凭据）。
 XFF（X-Forwarded-For）配置 #  如果 Easysearch 位于一个或多个代理或负载均衡器之后，xff 部分可用于配置从 HTTP 头中解析客户端真实 IP 地址的行为。这对于 IP 速率限制和代理认证尤为重要。"
---


# 后端配置

配置安全模块的第一步是确定如何验证用户身份。尽管 Easysearch 本身可以充当一个内部用户数据库，但许多人更喜欢集成企业现有的身份认证体系，例如 LDAP/Active Directory 服务器，或两者组合。设置身份验证和授权服务端的主要配置文件位于 `config/security/config.yml`。它定义了安全模块如何检索用户凭据、如何验证这些凭据以及如何从后端系统获取其他角色（可选）。

`config.yml` 主要包含三大部分：

```yml
config:
  dynamic:
    http: ...
    authc: ...
    authz: ...
```

## HTTP

`http` 部分具有以下配置项：

```yml
http:
  anonymous_auth_enabled: <true|false>
  xff:
    enabled: <true|false>
    internalProxies: '<regex pattern>'
    remoteIpHeader: '<header name>'
```

### 匿名认证

设置 `anonymous_auth_enabled` 可以选择是否开启匿名访问。如果启用，当请求中未提供任何凭据时，用户将以 `anonymous` 身份通过认证，并被分配 `anonymous_backendrole` 角色。如果禁用匿名身份验证，则至少在 `authc` 里面提供一个认证后端，否则安全模块将不予初始化。默认为 `false`。

> **提示：** 如果启用匿名认证，所有 HTTP 认证器将不会发起 challenge（即不会返回 401 要求客户端提供凭据）。

### XFF（X-Forwarded-For）配置

如果 Easysearch 位于一个或多个代理或负载均衡器之后，`xff` 部分可用于配置从 HTTP 头中解析客户端真实 IP 地址的行为。这对于 IP 速率限制和代理认证尤为重要。

| 参数 | 描述 | 默认值 |
|---|---|---|
| `enabled` | 是否启用 XFF 解析 | `false` |
| `internalProxies` | 可信内部代理 IP 地址的正则表达式。只有来自匹配 IP 的请求才会解析 XFF 头。例如 `192\.168\.0\.10\|192\.168\.0\.11`。设置为 `.*` 信任所有代理。 | 无 |
| `remoteIpHeader` | 包含客户端真实 IP 的 HTTP 请求头名称 | `x-forwarded-for` |

示例：

```yml
http:
  anonymous_auth_enabled: false
  xff:
    enabled: true
    internalProxies: '192\.168\.0\.10|192\.168\.0\.11'
    remoteIpHeader: 'x-forwarded-for'
```

## 认证（authc）

认证配置 `authc` 具有以下格式：

```yml
authc:
  <domain_name>:
    description: "<描述信息>"
    http_enabled: <true|false>
    transport_enabled: <true|false>
    order: <integer>
    http_authenticator:
      type: <type>
      challenge: <true|false>
      config:
        ...
    authentication_backend:
      type: <type>
      config:
        ...
```

配置 `authc` 里面的每一项被称为 _身份验证域（Authentication Domain）_。它指定了从何处获取用户凭据以及应针对哪个后端对它们进行身份验证。

您可以使用多个身份验证域。每个身份验证域都有一个名称（例如 `basic_internal_auth_domain`）、可选的 `description` 描述字段、`http_enabled`/`transport_enabled` 开关和排序参数 `order`。安全模块按 `order` 顺序依次使用它们。如果用户成功通过一个域进行了身份验证，安全模块将跳过剩余的验证域。

### 关于 Challenge

`challenge` 参数决定当未提供凭据时，HTTP 认证器是否应该向客户端发起认证挑战（如返回 HTTP 401 响应和 `WWW-Authenticate` 头）。默认值为 `true`。

> **注意：** 如果定义了多个 HTTP 认证器，请将不发起 challenge 的认证器（如 `proxy`、`clientcert`、`jwt`）放在前面，将发起 challenge 的认证器（如 `basic`）放在最后。因为无法同时用两种不同的认证方法向客户端发起 challenge。

### HTTP 认证器类型

`http_authenticator` 指定了在 HTTP 层上使用的身份验证方法。

| 类型 | 说明 | Challenge | 备注 |
|---|---|---|---|
| `basic` | HTTP 基本身份验证（用户名/密码） | 是 | 最常用的认证方式 |
| `proxy` | 代理认证，从 HTTP 请求头中提取用户信息 | 否 | 需要配合 XFF |
| `extended-proxy` | 扩展代理认证，支持自定义属性头 | 否 | 继承 `proxy` 的所有功能 |
| `clientcert` | 通过客户端 TLS 证书进行身份验证 | 否 | 需要 HTTPS |
| `kerberos` | 通过 Kerberos/SPNEGO 进行身份验证 | 是 | 需要企业功能 |
| `jwt` | 通过 JSON Web Token 进行身份验证 | 否 | 需要企业功能 |
| `openid` | 通过 OpenID Connect 进行身份验证 | 否 | 需要企业功能 |
| `saml` | 通过 SAML 进行身份验证 | 否 | 需要企业功能 |

#### Basic 认证

HTTP 基本身份验证是最简单也是最常用的认证方式。它从 HTTP `Authorization` 请求头中提取用户名和密码。

```yml
http_authenticator:
  type: basic
  challenge: true
```

无需额外配置。通常与 `internal` 或 `ldap` 认证后端配合使用。

#### 代理认证（Proxy）

代理认证用于当 Easysearch 位于一个已经完成用户认证的代理之后的场景。代理通过 HTTP 请求头将已认证的用户名和角色传递给 Easysearch。

> **注意：** 使用代理认证时，必须先启用 XFF（`xff.enabled: true`），并正确配置 `internalProxies` 以限定可信代理的 IP 范围。否则任何人都可以通过伪造请求头来冒充用户。

```yml
proxy_auth_domain:
  http_enabled: true
  transport_enabled: true
  order: 3
  http_authenticator:
    type: proxy
    challenge: false
    config:
      user_header: "x-proxy-user"
      roles_header: "x-proxy-roles"
      roles_separator: ","
  authentication_backend:
    type: noop
```

| 参数 | 描述 | 默认值 |
|---|---|---|
| `user_header` | 包含已认证用户名的 HTTP 请求头 | 无 |
| `roles_header` | 包含用户角色（逗号分隔）的 HTTP 请求头 | 无 |
| `roles_separator` | 角色分隔符 | `,` |

#### 扩展代理认证（Extended Proxy）

扩展代理认证继承了代理认证的所有功能，并额外支持通过 HTTP 请求头传递自定义用户属性。

```yml
http_authenticator:
  type: extended-proxy
  challenge: false
  config:
    user_header: "x-proxy-user"
    roles_header: "x-proxy-roles"
    attr_header_prefix: "x-proxy-ext-"
```

| 参数 | 描述 | 默认值 |
|---|---|---|
| `attr_header_prefix` | 自定义属性头的前缀。所有以此前缀开头的请求头都会被提取为用户属性。 | 无 |

例如，如果 `attr_header_prefix` 设置为 `x-proxy-ext-`，则请求头 `x-proxy-ext-department: engineering` 将为用户添加属性 `attr.proxy.department=engineering`。

#### 客户端证书认证（ClientCert）

通过客户端 TLS 证书进行身份验证。客户端证书必须受节点信任库中的根 CA 之一的信任。

```yml
clientcert_auth_domain:
  http_enabled: true
  transport_enabled: true
  order: 2
  http_authenticator:
    type: clientcert
    challenge: false
    config:
      username_attribute: cn
  authentication_backend:
    type: noop
```

| 参数 | 描述 | 默认值 |
|---|---|---|
| `username_attribute` | 从客户端证书 DN 中提取用户名所使用的属性。例如 `cn`、`ou` 等。如果不设置，则使用完整的 DN 作为用户名。 | 无（使用完整 DN） |
| `roles_attribute` | 从客户端证书 DN 中提取角色所使用的属性。 | 无 |

#### JWT 认证

通过 JSON Web Token（JWT）进行身份验证。JWT 通常由外部身份提供商签发，Easysearch 通过验证签名来确认用户身份。

```yml
jwt_auth_domain:
  http_enabled: true
  transport_enabled: true
  order: 0
  http_authenticator:
    type: jwt
    challenge: false
    config:
      signing_key: "base64 encoded HMAC key or public RSA/ECDSA pem key"
      jwt_header: "Authorization"
      jwt_url_parameter: null
      roles_key: null
      subject_key: null
  authentication_backend:
    type: noop
```

| 参数 | 描述 | 默认值 |
|---|---|---|
| `signing_key` | 用于验证 JWT 签名的密钥。对于 HMAC 使用 Base64 编码的密钥，对于 RSA/ECDSA 使用 PEM 格式的公钥。 | 无（必填） |
| `jwt_header` | 包含 JWT 的 HTTP 请求头名称 | `Authorization` |
| `jwt_url_parameter` | 如果 JWT 通过 URL 参数传递，指定参数名 | 无 |
| `roles_key` | JWT payload 中包含角色的字段名 | 无 |
| `subject_key` | JWT payload 中包含用户名的字段名 | 无（使用 `sub`） |

#### Kerberos 认证

通过 Kerberos/SPNEGO 协议进行身份验证，常用于与 Active Directory 集成的企业环境。

```yml
kerberos_auth_domain:
  http_enabled: true
  transport_enabled: true
  order: 6
  http_authenticator:
    type: kerberos
    challenge: true
    config:
      krb_debug: false
      strip_realm_from_principal: true
  authentication_backend:
    type: noop
```

| 参数 | 描述 | 默认值 |
|---|---|---|
| `krb_debug` | 是否输出 Kerberos/安全相关的调试日志到标准输出 | `false` |
| `strip_realm_from_principal` | 是否从 Kerberos principal 中去除 realm 部分。例如将 `user@REALM.COM` 简化为 `user`。 | `true` |

### 认证后端类型

`authentication_backend` 指定了对 HTTP 认证器提取的凭据进行验证的后端系统。

| 类型 | 说明 |
|---|---|
| `noop` | 不进行进一步的身份验证。适用于 HTTP 认证器已经完全验证了用户身份的情况，例如 JWT、Kerberos、客户端证书或代理认证。 |
| `internal` | 使用 `user.yml` 中定义的内部用户数据库进行认证。 |
| `ldap` | 使用 LDAP 或 Active Directory 进行认证。 |

#### Internal 认证后端

使用 Easysearch 内置的用户数据库（`config/security/user.yml`）进行认证。这是最简单的认证后端，适合用户数量较少或测试环境。

```yml
authentication_backend:
  type: internal
```

无需额外配置。

#### LDAP 认证后端

使用 LDAP 或 Active Directory 对用户进行身份验证。

```yml
ldap_auth_domain:
  http_enabled: true
  transport_enabled: true
  order: 1
  http_authenticator:
    type: basic
    challenge: true
  authentication_backend:
    type: ldap
    config:
      enable_ssl: false
      enable_start_tls: false
      enable_ssl_client_auth: false
      verify_hostnames: false
      hosts:
        - localhost:389
      bind_dn: "cn=admin,dc=example,dc=com"
      password: "password"
      userbase: 'ou=people,dc=example,dc=com'
      usersearch: '(uid={0})'
      username_attribute: null
```

**连接配置：**

| 参数 | 描述 | 默认值 |
|---|---|---|
| `hosts` | LDAP 服务器地址列表（`host:port` 格式） | 无（必填） |
| `bind_dn` | 用于连接 LDAP 服务器的绑定 DN | 无 |
| `password` | 绑定 DN 的密码 | 无 |
| `connect_timeout` | 连接超时时间（毫秒） | 无 |
| `response_timeout` | 响应超时时间（毫秒） | 无 |

**SSL/TLS 配置：**

| 参数 | 描述 | 默认值 |
|---|---|---|
| `enable_ssl` | 是否启用 LDAPS | `false` |
| `enable_start_tls` | 是否启用 StartTLS（`enable_ssl` 应为 `false`） | `false` |
| `enable_ssl_client_auth` | 是否发送客户端证书 | `false` |
| `verify_hostnames` | 是否验证 LDAP 服务器主机名 | `false` |
| `trust_all` | 是否信任所有 LDAP 服务器证书 | `false` |
| `cert_alias` | 客户端证书别名（JKS） | 无 |
| `ca_alias` | CA 证书别名（JKS） | 无 |
| `key_file` | 客户端私钥文件路径（PEM） | 无 |
| `cert_file` | 客户端证书文件路径（PEM） | 无 |
| `ca_file` | CA 证书文件路径（PEM） | 无 |
| `enabled_ssl_ciphers` | 允许的 SSL 加密套件 | 无 |
| `enabled_ssl_protocols` | 允许的 SSL 协议 | 无 |

**用户搜索配置：**

| 参数 | 描述 | 默认值 |
|---|---|---|
| `userbase` | 用户搜索的基础 DN | 无（必填） |
| `usersearch` | 搜索用户的 LDAP 过滤器。`{0}` 将被替换为用户名。 | 无（必填） |
| `username_attribute` | 使用用户条目中的此属性作为用户名。如果不设置则使用 DN。 | 无 |
| `search_all_bases` | 是否在所有基础 DN 下搜索 | `false` |

## 授权（authz）

对用户进行身份验证后，安全模块可以选择从后端权限系统收集用户的其他角色。

授权配置 `authz` 具有以下格式：

```yml
authz:
  <domain_name>:
    description: "<描述信息>"
    http_enabled: <true|false>
    transport_enabled: <true|false>
    authorization_backend:
      type: <type>
      config: ...
```

您可以在本节中定义多个条目。在授权阶段，执行顺序不相关，因此没有 `order` 字段。

### 授权后端类型

| 类型 | 说明 |
|---|---|
| `noop` | 跳过额外的角色收集。 |
| `ldap` | 从 LDAP 或 Active Directory 获取用户的后端角色。 |

#### LDAP 授权后端

从 LDAP 或 Active Directory 获取用户的角色信息。

```yml
authz:
  roles_from_ldap:
    http_enabled: true
    transport_enabled: true
    authorization_backend:
      type: ldap
      config:
        enable_ssl: false
        enable_start_tls: false
        enable_ssl_client_auth: false
        verify_hostnames: false
        hosts:
          - localhost:389
        bind_dn: "cn=admin,dc=example,dc=com"
        password: "password"
        rolebase: 'ou=groups,dc=example,dc=com'
        rolesearch: '(member={0})'
        userroleattribute: null
        userrolename: disabled
        rolename: cn
        resolve_nested_roles: true
        userbase: 'ou=people,dc=example,dc=com'
        usersearch: '(uid={0})'
```

LDAP 授权的连接和 SSL/TLS 配置与 [LDAP 认证后端](#ldap-认证后端) 相同。

**角色搜索配置：**

| 参数 | 描述 | 默认值 |
|---|---|---|
| `rolebase` | 角色搜索的基础 DN | 无（必填） |
| `rolesearch` | 搜索角色的 LDAP 过滤器。`{0}` 将被替换为用户的 DN，`{1}` 替换为用户名，`{2}` 替换为 `userroleattribute` 指定的属性值。 | 无（必填） |
| `userroleattribute` | 用户条目中包含角色信息的属性名。其值将替换 `rolesearch` 中的 `{2}`。 | 无 |
| `userrolename` | 用户条目中直接包含角色名的属性（如 `memberOf`）。设置为 `disabled` 禁用。 | `disabled` |
| `rolename` | 角色条目中包含角色名称的属性。也可以设置为 `dn` 使用完整 DN 作为角色名。 | `name` |
| `resolve_nested_roles` | 是否递归解析嵌套角色（角色的角色） | `true` |
| `skip_users` | 跳过匹配的用户，支持通配符和正则表达式 | 无 |
| `rolesearch_enabled` | 是否启用角色搜索 | `true` |
| `nested_role_filter` | 嵌套角色过滤器 | 无 |
| `max_nested_depth` | 最大嵌套角色解析深度 | 无 |
| `custom_attr_maxval_len` | 自定义属性最大值长度 | 无 |
| `custom_attr_whitelist` | 允许的自定义属性白名单 | 无 |

## 认证失败监听器（auth_failure_listeners）

安全模块支持配置认证失败监听器，用于实现基于 IP 或用户名的速率限制，防止暴力破解攻击。

```yml
config:
  dynamic:
    auth_failure_listeners:
      ip_rate_limiting:
        type: ip
        allowed_tries: 10
        time_window_seconds: 3600
        block_expiry_seconds: 600
        max_blocked_clients: 100000
        max_tracked_clients: 100000
      internal_authentication_backend_limiting:
        type: username
        authentication_backend: internal
        allowed_tries: 10
        time_window_seconds: 3600
        block_expiry_seconds: 600
        max_blocked_clients: 100000
        max_tracked_clients: 100000
```

### IP 速率限制

基于 IP 地址的速率限制，无论使用哪个用户名，只要某个 IP 地址在指定时间窗口内认证失败次数超过阈值，就会被临时封禁。

| 参数 | 描述 | 默认值 |
|---|---|---|
| `type` | 设置为 `ip` | 无（必填） |
| `allowed_tries` | 在被封禁前允许的失败尝试次数 | `10` |
| `time_window_seconds` | 计数失败尝试的时间窗口（秒） | `3600` |
| `block_expiry_seconds` | 封禁持续时间（秒） | `600` |
| `max_blocked_clients` | 最大被封禁客户端数量 | `100000` |
| `max_tracked_clients` | 最大追踪客户端数量 | `100000` |

### 用户名速率限制

基于用户名的速率限制，针对特定认证后端的用户名进行限制。

| 参数 | 描述 | 默认值 |
|---|---|---|
| `type` | 设置为 `username` | 无（必填） |
| `authentication_backend` | 要限制的认证后端名称（如 `internal`） | 无（必填） |
| `allowed_tries` | 在被封禁前允许的失败尝试次数 | `10` |
| `time_window_seconds` | 计数失败尝试的时间窗口（秒） | `3600` |
| `block_expiry_seconds` | 封禁持续时间（秒） | `600` |
| `max_blocked_clients` | 最大被封禁客户端数量 | `100000` |
| `max_tracked_clients` | 最大追踪客户端数量 | `100000` |

## 完整配置示例

以下是一个包含多种认证方式的完整配置示例：

```yml
_meta:
  type: "config"
  config_version: 2

config:
  dynamic:
    http:
      anonymous_auth_enabled: false
      xff:
        enabled: true
        internalProxies: '192\.168\.0\.10|192\.168\.0\.11'
        remoteIpHeader: 'x-forwarded-for'
    authc:
      basic_internal_auth_domain:
        description: "Authenticate via HTTP Basic against internal users database"
        http_enabled: true
        transport_enabled: true
        order: 4
        http_authenticator:
          type: basic
          challenge: true
        authentication_backend:
          type: internal
      proxy_auth_domain:
        description: "Authenticate via proxy"
        http_enabled: false
        transport_enabled: false
        order: 3
        http_authenticator:
          type: proxy
          challenge: false
          config:
            user_header: "x-proxy-user"
            roles_header: "x-proxy-roles"
        authentication_backend:
          type: noop
      jwt_auth_domain:
        description: "Authenticate via Json Web Token"
        http_enabled: false
        transport_enabled: false
        order: 0
        http_authenticator:
          type: jwt
          challenge: false
          config:
            signing_key: "base64 encoded HMAC key or public RSA/ECDSA pem key"
            jwt_header: "Authorization"
            jwt_url_parameter: null
            roles_key: null
            subject_key: null
        authentication_backend:
          type: noop
      clientcert_auth_domain:
        description: "Authenticate via SSL client certificates"
        http_enabled: false
        transport_enabled: false
        order: 2
        http_authenticator:
          type: clientcert
          challenge: false
          config:
            username_attribute: cn
        authentication_backend:
          type: noop
      ldap_auth_domain:
        description: "Authenticate via LDAP or Active Directory"
        http_enabled: false
        transport_enabled: false
        order: 1
        http_authenticator:
          type: basic
          challenge: true
        authentication_backend:
          type: ldap
          config:
            enable_ssl: false
            enable_start_tls: false
            enable_ssl_client_auth: false
            verify_hostnames: false
            hosts:
              - localhost:389
            bind_dn: "cn=admin,dc=example,dc=com"
            password: "password"
            userbase: 'ou=people,dc=example,dc=com'
            usersearch: '(uid={0})'
            username_attribute: null
    authz:
      basic_authz:
        http_enabled: true
        transport_enabled: true
        authorization_backend:
          type: noop
      roles_from_ldap:
        description: "Authorize via LDAP or Active Directory"
        http_enabled: false
        transport_enabled: false
        authorization_backend:
          type: ldap
          config:
            enable_ssl: false
            enable_start_tls: false
            enable_ssl_client_auth: false
            verify_hostnames: false
            hosts:
              - localhost:389
            bind_dn: "cn=admin,dc=example,dc=com"
            password: "password"
            rolebase: 'ou=groups,dc=example,dc=com'
            rolesearch: '(member={0})'
            userroleattribute: null
            userrolename: disabled
            rolename: cn
            resolve_nested_roles: true
            userbase: 'ou=people,dc=example,dc=com'
            usersearch: '(uid={0})'
    auth_failure_listeners:
      ip_rate_limiting:
        type: ip
        allowed_tries: 10
        time_window_seconds: 3600
        block_expiry_seconds: 600
        max_blocked_clients: 100000
        max_tracked_clients: 100000
      internal_authentication_backend_limiting:
        type: username
        authentication_backend: internal
        allowed_tries: 10
        time_window_seconds: 3600
        block_expiry_seconds: 600
        max_blocked_clients: 100000
        max_tracked_clients: 100000
```

