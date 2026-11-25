---
title: "用户与角色"
date: 0001-01-01
summary: "用户与角色 #  安全模块包括一个内部用户数据库。可以使用此数据库代替外部身份验证系统（如 LDAP 或 Active Directory），或作为外部身份验证系统的补充。
角色是控制对集群访问的核心方式。角色包含集群范围权限、特定于索引的权限、文档和字段级安全性以及租户的任意组合。然后，将用户映射到这些角色，以便用户获得这些权限。
除非您需要创建新的只读或隐藏用户，我们强烈建议使用 REST API 来创建新的用户、角色和角色映射。.yml 文件更适合初始设置，而不是持续维护。
相关指南（先读这些） #   安全与多租户最佳实践 权限控制总览  创建用户 #  user.yml #  参照本地文件配置。
REST API #  参照创建用户。
创建角色 #  role.yml #  参照本地文件配置。
REST API #  参照创建角色。
 常见注意事项：如果一个角色需要执行常见的文档写入 REST API，例如 PUT /&lt;index&gt;/_doc/&lt;id&gt;、POST /&lt;index&gt;/_bulk、POST /&lt;index&gt;/_update/&lt;id&gt; 或 DELETE /&lt;index&gt;/_doc/&lt;id&gt;，当前实现下通常不仅要在 indices[*].privileges 中授予 write / index / delete 等索引级权限，还需要在 cluster 中额外授予 cluster_composite_ops。详细说明见权限列表。
 内置索引角色 #  以下角色由安全模块内置。默认开启静态资源加载时，升级到包含这些角色的版本并完成相关节点更新后，无需重新导入 role."
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

> 常见注意事项：如果一个角色需要执行常见的文档写入 REST API，例如 `PUT /<index>/_doc/<id>`、`POST /<index>/_bulk`、`POST /<index>/_update/<id>` 或 `DELETE /<index>/_doc/<id>`，当前实现下通常不仅要在 `indices[*].privileges` 中授予 `write` / `index` / `delete` 等索引级权限，还需要在 `cluster` 中额外授予 `cluster_composite_ops`。详细说明见[权限列表](./permissions.md#索引级别)。

## 内置索引角色

以下角色由安全模块内置。默认开启静态资源加载时，升级到包含这些角色的版本并完成相关节点更新后，无需重新导入 `role.yml` 即可使用角色定义；仍需管理员显式将用户映射到角色，不会自动增加用户权限。

| 角色 | 索引名称范围 | 权限 |
| --- | --- | --- |
| `readonly` | `*` | 现有的通用数据读取权限 |
| `data_readonly` | `/[^.].*/` | 按名称读取非 dot 开头的索引或别名，不含文档写入或索引管理 |
| `user_prefix_readonly` | `${user.name}-*` | 读取当前用户名对应范围，不含文档写入或索引管理 |
| `user_prefix_manager` | `${user.name}-*` | 管理当前用户名对应范围，包括创建、删除索引及修改 mapping/settings，不含文档读写 |

用户名为 `china` 时，`${user.name}-*` 替换为 `china-*`，可匹配 `china-orders`，不匹配 `china` 或 `china2-orders`。用户名直接参与模式匹配，应适用于能够作为确定索引前缀的命名；例如小写字母或数字开头，后续使用小写字母、数字、下划线或连字符。用户名含 `*`、`?` 等模式字符时，不要用这类角色承诺用户名的字面隔离，应配置明确的业务前缀角色。系统不会为这些角色自动转换用户名大小写或转义模式字符。

**名称范围包含现有别名授权语义。**符合模式的别名会将其真实索引目标纳入授权，目标名称不必满足原模式。例如管理员将 `china-bridge` 指向 `usa-orders`，`user_prefix_readonly` 或 `user_prefix_manager` 可获得相应目标权限。因此这些角色不保证底层真实索引名称严格隔离。

`data_readonly` 同样只按名称模式排除 dot 前缀，不能作为显式拒绝规则：普通名称的别名仍可能授权到 dot 开头的真实索引；dot 名称的别名指向已授权普通索引时也可能被允许。该角色不自动排除所有非 dot 系统索引，现有安全模块保护继续生效。它不等同于“禁止访问所有系统索引”的角色，也不提供完整集群监控或管理 UI 浏览权限。

`user_prefix_manager` 能删除索引及其中的全部数据，请按管理职责分配。需要读取并管理本人模式范围时，可同时分配两个 `user_prefix_*` 角色；这不授予文档写入权限。多个角色不会相互收窄授权，不要通过叠加 `data_readonly` 与用户名角色来实现两个范围的交集。

静态角色定义不可通过普通安全管理员修改或删除。升级前应检查同名自定义角色：已存角色与静态角色重名会导致动态配置加载报错，不能依赖角色 API 合并视图的显示结果判断冲突已解决。主动关闭静态资源加载的部署不会自动获得这些定义。

## 内置运维与配置角色

这些静态角色同样需要管理员显式分配，加载和升级冲突规则与内置索引角色一致。它们不授予业务文档读取或写入权限。

| 角色 | 权限与适用范围 |
| --- | --- |
| `monitoring_collector` | 集群和所有索引的监控，复用 `cluster_monitor`、`indices_monitor`；不含索引管理 |
| `ingest_readonly` | 查询 legacy index template、composable index template、component template 和 ingest pipeline；不含模拟执行 |
| `ingest_admin` | 全局管理上述模板和 ingest pipeline，包含相应权限组内的模拟操作；不含 search pipeline 管理 |
| `snapshot_readonly` | 全局查询仓库、快照及快照状态；不能创建、恢复、删除或克隆快照，不能管理仓库 |
| `snapshot_user` | 在 `snapshot_readonly` 基础上允许创建快照；不能恢复、删除或克隆快照，不能注册、删除、验证或清理仓库 |

`monitoring_collector` 面向可信采集器或运维人员，可能看到索引名称、集群配置、节点和线程等敏感运维信息；不承诺业务索引名称隔离。现有 `monitor` 角色保持原有权限，不会因为新增采集角色自动获得索引监控权限。

两个 ingest 角色具有全局配置职责，可以看到系统模板。它们不会自动获得模板匹配索引的文档权限，也不涉及前端页面权限。

两个 snapshot 角色可以看到仓库和快照中的索引清单。`snapshot_user` 是集群级备份信任角色，可以备份自身无权搜索的索引，不提供按当前用户数据读取范围限制备份的保证；现有内部索引保护继续生效。仓库需由管理员预先注册。快照策略管理属于独立权限范围，不随这两个角色授予。

## 内置功能只读角色

安装并启用对应模块时，可分配以下静态角色。它们用于查看策略、任务及状态，不授予业务文档读写权限，也不会自动授予功能管理权限。

| 角色 | 可查询的内容 | 不包含的权限 |
| --- | --- | --- |
| `ilm_readonly` | 生命周期策略列表、策略详情及托管索引状态 | 创建或修改策略、绑定、更换、移除策略及重试任务 |
| `rollup_readonly` | Rollup任务列表、配置、状态及全局统计 | 创建、修改、删除、启动或停止任务；业务汇总搜索 |
| `snapshot_management_readonly` | 自动快照策略列表、配置及状态 | 创建或修改策略、启停调度、创建快照及仓库管理 |
| `replication_readonly` | 全部复制状态、单索引复制状态及leader/follower/autofollow统计 | 启停或修改复制、自动跟随配置、文档读取以及changes/file_chunk数据传输 |

这些角色面向可信运维人员，可能看到索引、仓库、任务及连接名称等配置，不提供新的租户隔离机制。ILM、Rollup和自动快照配置查询仍受各模块现有backend-role过滤设置约束；开启过滤时，角色本身不会绕过资源所属backend role的检查。Rollup统计保留全局可见性，不受配置列表的backend-role过滤限制。复制状态和统计同样属于全局运维信息。当前复制统计请通过 REST API 使用；Transport 客户端统计响应存在已知反序列化问题。

`rollup_readonly` 的任务列表查询不等同于汇总索引的数据查询。需要搜索业务数据时另行分配相应的数据角色。`snapshot_management_readonly` 只查看自动策略，与查看实际仓库和快照的 `snapshot_readonly` 是不同职责，需要时由管理员显式组合分配。

## 映射用户到角色

### role_mapping.yml

参照[本地文件配置](../configuration/yaml.md#role_mappingyml)。

### REST API

参照[创建角色映射](./api.md#创建角色映射)。

### 内置角色分配示例

假设已创建用户 `china`，需要让其读取 `china-*` 范围。管理员先检查内置角色定义：

```text
GET /_security/role/user_prefix_readonly
```

然后将用户映射到该角色，无需重新创建角色定义：

```json
PUT /_security/role_mapping/user_prefix_readonly
{
  "users": ["china"]
}
```

`PUT` 会替换该角色的整个映射。若已有映射，应先通过 `GET /_security/role_mapping/user_prefix_readonly` 获取现有内容，保留其他用户、`external_roles` 和 `hosts`，或使用[修改角色映射](./api.md#修改角色映射)接口追加用户。

以 `china` 的身份认证后，检查账户返回的 `roles` 包含 `user_prefix_readonly`，再查询已存在的索引：

```text
GET /_security/account
GET /china-orders/_search
```

验证用户权限时应使用该用户的凭据。其他内置角色也通过同一映射接口分配，将路径中的角色名替换为所需角色即可；角色选择和组合应遵循上面的权限范围说明。

