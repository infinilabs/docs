---
title: "用户与角色"
date: 0001-01-01
summary: "用户与角色 #  安全模块包括一个内部用户数据库。可以使用此数据库代替外部身份验证系统（如 LDAP 或 Active Directory），或作为外部身份验证系统的补充。
角色是控制对集群访问的核心方式。角色包含集群范围权限、特定于索引的权限、文档和字段级安全性以及租户的任意组合。然后，将用户映射到这些角色，以便用户获得这些权限。
除非您需要创建新的 只读或隐藏用户，我们强烈建议使用 REST API 来创建新的用户、角色和角色映射。.yml 文件更适合初始设置，而不是持续维护。
相关指南（先读这些） #    安全与多租户最佳实践  权限控制总览  创建用户 #  user.yml #  参照 本地文件配置。
REST API #  参照 创建用户。
创建角色 #  role.yml #  参照 本地文件配置。
REST API #  参照 创建角色。
映射用户到角色 #  role_mapping.yml #  参照 本地文件配置。
REST API #  参照 创建角色映射。"
---


# 用户与角色

安全模块包括一个内部用户数据库。可以使用此数据库代替外部身份验证系统（如 LDAP 或 Active Directory），或作为外部身份验证系统的补充。

角色是控制对集群访问的核心方式。角色包含集群范围权限、特定于索引的权限、文档和字段级安全性以及租户的任意组合。然后，将用户映射到这些角色，以便用户获得这些权限。

除非您需要创建新的[只读或隐藏用户](./api.md#隐藏的保留资源)，我们强烈建议使用 REST API 来创建新的用户、角色和角色映射。`.yml` 文件更适合初始设置，而不是持续维护。

## 相关指南（先读这些）

- [安全与多租户最佳实践]({{< relref "/docs/best-practices/security-and-multi-tenancy.md" >}})
- [权限控制总览]({{< relref "/docs/operations/security/access-control/_index.md" >}})

## 创建用户

### user.yml

参照[本地文件配置](../configuration/yaml.md#useryml)。

### REST API

参照[创建用户](./api.md#创建用户)。

## 创建角色

### role.yml

参照[本地文件配置](../configuration/yaml.md#roleyml)。

### REST API

参照[创建角色](./api.md#创建一个角色)。

## 映射用户到角色

### role_mapping.yml

参照[本地文件配置](../configuration/yaml.md#role_mappingyml)。

### REST API

参照[创建角色映射](./api.md#创建角色映射)。

