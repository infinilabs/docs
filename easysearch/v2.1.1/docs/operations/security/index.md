---
title: "安全配置"
date: 0001-01-01
description: "认证授权、TLS 加密、访问控制、审计日志——保护集群数据安全。"
summary: "安全配置 #  Easysearch 内置完善的安全能力，帮助保护集群中的重要数据。本模块涵盖从启用安全到精细权限控制的完整内容。
 安全能力概览 #     功能 说明     传输加密 基于 TLS 的节点间与客户端通信加密   身份认证 HTTP 基础认证、LDAP、OIDC、代理认证等   角色权限控制 基于角色的访问控制（RBAC）   细粒度权限 索引级、文档级、字段级安全控制   审计日志 记录安全相关事件，支持合规审计     快速启用安全 #  最小闭环步骤 #   启用安全功能：在 easysearch.yml 中开启安全模块 配置管理账号：创建运维和系统账号 开启 TLS 加密：配置节点间和客户端通信加密 开启审计日志：记录关键安全事件 验证权限：测试各账号的权限边界  安全检查清单 #   安全模块已启用 已配置管理账号和系统账号 已定义基础角色体系 集群内外通信均启用 TLS 端口暴露最小化，防火墙已配置 审计日志已开启   认证 #  支持的认证方式 #   本地用户（YAML 配置） LDAP / Active Directory OIDC / SAML 代理认证（通过网关/反向代理）  实践建议 #   复用现有身份系统：优先接入企业 SSO/LDAP 区分账号类型：人类账号与技术账号分开管理 最小权限原则：先收紧再按需放宽  详细配置见 认证后端配置。"
---


# 安全配置

Easysearch 内置完善的安全能力，帮助保护集群中的重要数据。本模块涵盖从启用安全到精细权限控制的完整内容。

---

## 安全能力概览

| 功能 | 说明 |
|------|------|
| **传输加密** | 基于 TLS 的节点间与客户端通信加密 |
| **身份认证** | HTTP 基础认证、LDAP、OIDC、代理认证等 |
| **角色权限控制** | 基于角色的访问控制（RBAC） |
| **细粒度权限** | 索引级、文档级、字段级安全控制 |
| **审计日志** | 记录安全相关事件，支持合规审计 |

---

## 快速启用安全

### 最小闭环步骤

1. **启用安全功能**：在 `easysearch.yml` 中开启安全模块
2. **配置管理账号**：创建运维和系统账号
3. **开启 TLS 加密**：配置节点间和客户端通信加密
4. **开启审计日志**：记录关键安全事件
5. **验证权限**：测试各账号的权限边界

### 安全检查清单

- [ ] 安全模块已启用
- [ ] 已配置管理账号和系统账号
- [ ] 已定义基础角色体系
- [ ] 集群内外通信均启用 TLS
- [ ] 端口暴露最小化，防火墙已配置
- [ ] 审计日志已开启

---

## 认证

### 支持的认证方式

- 本地用户（YAML 配置）
- LDAP / Active Directory
- OIDC / SAML
- 代理认证（通过网关/反向代理）

### 实践建议

- **复用现有身份系统**：优先接入企业 SSO/LDAP
- **区分账号类型**：人类账号与技术账号分开管理
- **最小权限原则**：先收紧再按需放宽

详细配置见 [认证后端配置]({{< relref "./configuration/backend.md" >}})。

---

## 授权

Easysearch 采用**基于角色的访问控制（RBAC）**：

```
用户 ──▶ 角色 ──▶ 权限
         │
         ├── 集群权限（cluster:*）
         ├── 索引权限（indices:*）
         └── 文档/字段级权限
```

### 典型角色设计

| 角色 | 权限范围 | 适用场景 |
|------|----------|----------|
| 只读分析 | 指定索引的 `read` | 数据分析师 |
| 写入 | 指定索引的 `write` | 数据采集服务 |
| 索引管理 | `indices:admin/*` | 运维人员 |
| 集群管理 | `cluster:admin/*` | 平台管理员 |

详细配置见 [用户与角色]({{< relref "./access-control/users-roles.md" >}})。

---

## 传输加密

### TLS 配置要点

| 层级 | 用途 | 推荐配置 |
|------|------|----------|
| Transport | 节点间通信 | 必须启用 |
| HTTP | 客户端访问 | 生产环境必须启用 |

### 证书规划

- 区分生产/预发/测试环境的证书
- 使用企业 CA 或自签 CA
- 规划证书轮换流程

详细配置见 [TLS 配置]({{< relref "./configuration/tls.md" >}})。

---

## 细粒度权限控制

### 索引级安全

限制用户只能访问特定索引：

```yaml
roles:
  logs_reader:
    index_permissions:
      - index_patterns: ["logs-*"]
        allowed_actions: ["read"]
```

### 文档级安全

限制用户只能看到特定文档：

```yaml
roles:
  dept_a_reader:
    index_permissions:
      - index_patterns: ["*"]
        dls: '{"term": {"department": "A"}}'
```

### 字段级安全

限制用户只能看到特定字段：

```yaml
roles:
  masked_reader:
    index_permissions:
      - index_patterns: ["*"]
        fls: ["~password", "~credit_card"]  # 排除敏感字段
```

详细说明见：
- [文档级安全]({{< relref "./access-control/document-level-security.md" >}})
- [字段级安全]({{< relref "./access-control/field-level-security.md" >}})
- [字段脱敏]({{< relref "./access-control/field-masking.md" >}})

---

## 审计日志

### 开启审计

审计日志记录安全相关事件，便于追溯和合规：

- 登录成功/失败
- 权限拒绝
- 高危操作（删除索引、修改安全配置）

### 实践建议

- 将审计日志发送到独立的集中日志平台
- 对高风险行为配置告警
- 按合规要求设置保留期

---

## 模块文档

### 访问控制

| 文档 | 说明 |
|------|------|
| [用户与角色]({{< relref "./access-control/users-roles.md" >}}) | 用户、角色的创建与映射 |
| [权限列表]({{< relref "./access-control/permissions.md" >}}) | 可用权限的完整清单 |
| [文档级安全]({{< relref "./access-control/document-level-security.md" >}}) | 按条件限制可见文档 |
| [字段级安全]({{< relref "./access-control/field-level-security.md" >}}) | 限制可见字段 |
| [字段脱敏]({{< relref "./access-control/field-masking.md" >}}) | 敏感字段脱敏显示 |
| [跨集群搜索]({{< relref "./access-control/cross-cluster-search.md" >}}) | 跨集群搜索的权限配置 |
| [Run As]({{< relref "./access-control/run-as.md" >}}) | 以其他用户身份执行 |
| [安全 API]({{< relref "./access-control/api.md" >}}) | 安全相关的 REST API |

### 配置

| 文档 | 说明 |
|------|------|
| [YAML 配置]({{< relref "./configuration/yaml.md" >}}) | 本地用户和角色配置 |
| [认证后端]({{< relref "./configuration/backend.md" >}}) | LDAP、OIDC 等认证配置 |
| [TLS 配置]({{< relref "./configuration/tls.md" >}}) | 传输层加密配置 |
| [系统索引]({{< relref "./configuration/system-indices.md" >}}) | 安全相关系统索引 |

---

## 相关文档

- [安全与多租户最佳实践]({{< relref "/docs/best-practices/security-and-multi-tenancy" >}})：架构设计指南
- [故障排查]({{< relref "../troubleshooting" >}})：安全相关问题诊断
