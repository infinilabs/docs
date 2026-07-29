---
title: "接入企业认证体系"
date: 0001-01-01
description: "将 Easysearch 纳入现有 SSO/LDAP 等身份认证体系的整体方案与配置要点。"
summary: "接入企业认证体系 #  在企业环境中，Easysearch 通常需要与已有的身份认证系统集成，实现统一的用户管理和单点登录。本文介绍常见的集成模式与配置要点。
相关指南 #    安全 API 概述  用户与角色管理  多租户与权限模型  认证模式概览 #     模式 适用场景 复杂度 说明     内置用户 小型团队、开发测试 低 直接在 Easysearch 中管理用户和密码   LDAP/AD 已有 Active Directory 中 对接企业目录服务，集中管理用户   OIDC/OAuth 2.0 SSO 场景、云原生架构 中 对接 Keycloak、Auth0、Azure AD 等   反向代理 + Header 已有统一认证网关 低 由网关完成认证，Header 传递身份信息   SAML 2.0 企业级 SSO 高 对接 ADFS、Okta 等 SAML IdP    LDAP/Active Directory 集成 #  LDAP 是最常见的企业认证集成方式，将 Easysearch 的用户验证委派给现有的 LDAP 或 Active Directory。"
---


# 接入企业认证体系

在企业环境中，Easysearch 通常需要与已有的身份认证系统集成，实现统一的用户管理和单点登录。本文介绍常见的集成模式与配置要点。

## 相关指南

- [安全 API 概述]({{< relref "../../operations/security/" >}})
- [用户与角色管理]({{< relref "../../operations/security/access-control/users-roles.md" >}})
- [多租户与权限模型]({{< relref "./multi-tenant-access-control.md" >}})

## 认证模式概览

| 模式             | 适用场景               | 复杂度 | 说明                                     |
| ---------------- | ---------------------- | ------ | ---------------------------------------- |
| 内置用户         | 小型团队、开发测试     | 低     | 直接在 Easysearch 中管理用户和密码       |
| LDAP/AD          | 已有 Active Directory  | 中     | 对接企业目录服务，集中管理用户           |
| OIDC/OAuth 2.0   | SSO 场景、云原生架构   | 中     | 对接 Keycloak、Auth0、Azure AD 等        |
| 反向代理 + Header | 已有统一认证网关       | 低     | 由网关完成认证，Header 传递身份信息      |
| SAML 2.0         | 企业级 SSO             | 高     | 对接 ADFS、Okta 等 SAML IdP             |

## LDAP/Active Directory 集成

LDAP 是最常见的企业认证集成方式，将 Easysearch 的用户验证委派给现有的 LDAP 或 Active Directory。

### 配置示例

在 `easysearch-security` 配置中添加 LDAP 认证后端：

```yaml
# config/security/config.yml
_meta:
  type: "config"
  config_version: 2

config:
  dynamic:
    authc:
      ldap_auth:
        http_enabled: true
        transport_enabled: true
        order: 1
        http_authenticator:
          type: basic
          challenge: true
        authentication_backend:
          type: ldap
          config:
            hosts:
              - ldap.example.com:636
            enable_ssl: true
            bind_dn: "cn=admin,dc=example,dc=com"
            password: "admin_password"
            userbase: "ou=users,dc=example,dc=com"
            usersearch: "(uid={0})"
```

### 角色映射

将 LDAP 组映射到 Easysearch 角色：

```yaml
# config/security/roles_mapping.yml
admin_role:
  backend_roles:
    - "cn=es-admins,ou=groups,dc=example,dc=com"

readonly_role:
  backend_roles:
    - "cn=es-viewers,ou=groups,dc=example,dc=com"
```

## OIDC 集成

OpenID Connect 适用于与 Keycloak、Azure AD、Okta 等 IdP 对接的 SSO 场景。

### 典型流程

```
用户 → 浏览器 → Easysearch Dashboards
                     ↓ 302 重定向
              OIDC IdP（登录页面）
                     ↓ Authorization Code
              Easysearch（Token 验证）
                     ↓
              返回数据 / Dashboards 页面
```

### 关键配置项

| 配置项             | 说明                     | 示例                                        |
| ------------------ | ------------------------ | ------------------------------------------- |
| `openid_connect_url` | IdP 的 Discovery URL   | `https://keycloak/realms/es/.well-known/...`|
| `client_id`        | 注册的客户端 ID          | `easysearch-client`                         |
| `client_secret`    | 客户端密钥               | `xxxxxxxx`                                  |
| `subject_key`      | JWT 中的用户标识字段     | `preferred_username`                        |
| `roles_key`        | JWT 中的角色字段         | `realm_access.roles`                        |

## 反向代理认证

当企业已有统一认证网关（如 Nginx + OAuth2 Proxy、APISIX、Kong）时，可以通过 HTTP Header 传递已认证的用户身份。

### 架构

```
用户 → 认证网关（完成认证）→ 注入 X-Proxy-User Header → Easysearch
```

### 配置要点

```yaml
proxy_auth:
  http_enabled: true
  transport_enabled: false
  order: 0
  http_authenticator:
    type: proxy
    challenge: false
    config:
      user_header: "X-Proxy-User"
      roles_header: "X-Proxy-Roles"
  authentication_backend:
    type: noop
```

> **安全提示**：务必确保 Easysearch 仅接受来自可信代理的请求，建议通过网络策略或 IP 白名单限制直接访问。

## 安全实践建议

| 实践                 | 说明                                                 |
| -------------------- | ---------------------------------------------------- |
| 启用 HTTPS           | 所有认证流程必须走 TLS 加密通道                       |
| 最小权限原则         | 角色映射时只授予必要的索引和操作权限                  |
| 审计日志             | 开启审计日志记录认证成功/失败事件                     |
| 密钥轮换             | 定期更换 LDAP Bind 密码和 OIDC Client Secret         |
| 失败锁定             | 配置连续认证失败后的账户锁定策略                      |


