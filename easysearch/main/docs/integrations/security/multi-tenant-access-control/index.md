---
title: "多租户与权限模型实践"
date: 0001-01-01
description: "结合 Easysearch 的索引/文档/字段级安全能力，设计多租户与权限模型。"
summary: "多租户与权限模型实践 #  在 SaaS 平台或企业多部门共享 Easysearch 集群的场景下，需要一套完善的多租户隔离与权限控制方案。Easysearch 提供了索引级、文档级和字段级三层安全能力来支撑这一需求。
相关指南 #    用户与角色管理  接入企业认证体系  多租户隔离模式 #  模式一：按索引隔离 #  每个租户使用独立的索引（或索引前缀），通过角色控制访问范围。
tenant_a_orders tenant_a_products tenant_b_orders tenant_b_products 角色定义示例：
PUT _security/role/tenant_a_role { &#34;cluster_permissions&#34;: [&#34;cluster_composite_ops_ro&#34;], &#34;index_permissions&#34;: [ { &#34;index_patterns&#34;: [&#34;tenant_a_*&#34;], &#34;allowed_actions&#34;: [&#34;crud&#34;, &#34;create_index&#34;] } ] }    优点 缺点     隔离彻底，互不影响 索引数量随租户增长   性能可独立调优 跨租户查询需要额外聚合   易于理解和调试 集群管理复杂度较高    模式二：按字段标记隔离 #  所有租户共享索引，通过 tenant_id 字段区分，利用文档级安全（DLS）实现隔离。"
---


# 多租户与权限模型实践

在 SaaS 平台或企业多部门共享 Easysearch 集群的场景下，需要一套完善的多租户隔离与权限控制方案。Easysearch 提供了索引级、文档级和字段级三层安全能力来支撑这一需求。

## 相关指南

- [用户与角色管理]({{< relref "../../operations/security/access-control/users-roles.md" >}})
- [接入企业认证体系]({{< relref "./auth-integration.md" >}})

## 多租户隔离模式

### 模式一：按索引隔离

每个租户使用独立的索引（或索引前缀），通过角色控制访问范围。

```
tenant_a_orders
tenant_a_products
tenant_b_orders
tenant_b_products
```

**角色定义示例**：

```json
PUT _security/role/tenant_a_role
{
  "cluster_permissions": ["cluster_composite_ops_ro"],
  "index_permissions": [
    {
      "index_patterns": ["tenant_a_*"],
      "allowed_actions": ["crud", "create_index"]
    }
  ]
}
```

| 优点                   | 缺点                        |
| ---------------------- | --------------------------- |
| 隔离彻底，互不影响     | 索引数量随租户增长           |
| 性能可独立调优         | 跨租户查询需要额外聚合       |
| 易于理解和调试         | 集群管理复杂度较高           |

### 模式二：按字段标记隔离

所有租户共享索引，通过 `tenant_id` 字段区分，利用文档级安全（DLS）实现隔离。

```json
PUT _security/role/tenant_a_dls_role
{
  "cluster_permissions": ["cluster_composite_ops_ro"],
  "index_permissions": [
    {
      "index_patterns": ["shared_orders"],
      "dls": "{\"term\": {\"tenant_id\": \"tenant_a\"}}",
      "allowed_actions": ["read"]
    }
  ]
}
```

| 优点                   | 缺点                        |
| ---------------------- | --------------------------- |
| 索引数量少，管理简单   | DLS 对查询性能有一定影响     |
| 跨租户聚合方便         | 隔离不如独立索引彻底         |
| 适合租户数量多的场景   | 需要确保写入时带 tenant_id   |

### 模式对比

| 维度       | 按索引隔离 | 按字段隔离 |
| ---------- | ---------- | ---------- |
| 隔离强度   | 强         | 中         |
| 管理复杂度 | 高         | 低         |
| 性能影响   | 无额外开销 | DLS 过滤开销 |
| 适用租户数 | <100       | 100+       |
| 推荐场景   | 合规要求高 | SaaS 平台  |

## 文档级安全（DLS）

DLS 允许在角色级别定义查询过滤条件，用户只能看到满足条件的文档。

```json
{
  "index_permissions": [
    {
      "index_patterns": ["logs-*"],
      "dls": "{\"bool\": {\"must\": [{\"term\": {\"department\": \"engineering\"}}]}}",
      "allowed_actions": ["read"]
    }
  ]
}
```

## 字段级安全（FLS）

FLS 控制用户能看到文档中的哪些字段，适用于敏感数据保护。

```json
{
  "index_permissions": [
    {
      "index_patterns": ["employees"],
      "fls": ["name", "department", "title"],
      "allowed_actions": ["read"]
    }
  ]
}
```

上例中，用户只能看到 `name`、`department`、`title` 三个字段，`salary`、`ssn` 等敏感字段被隐藏。

也可以使用排除模式：

```json
{
  "fls": ["~salary", "~ssn"]
}
```

## 权限设计最佳实践

| 实践                   | 说明                                                       |
| ---------------------- | ---------------------------------------------------------- |
| 角色分层               | 定义基础角色（只读、读写）+ 业务角色（订单管理、日志查看）  |
| 最小权限               | 每个角色只授予完成工作所需的最小索引和操作权限              |
| 索引模式通配符         | 使用 `tenant_*_logs` 等模式批量授权，避免逐索引配置        |
| 分离管理员与业务用户    | 集群管理操作（`cluster_all`）只授予运维角色                |
| 审计日志               | 开启审计日志，记录敏感操作便于事后追溯                      |
| 定期审查               | 定期审查角色映射，移除不再需要的权限                        |


