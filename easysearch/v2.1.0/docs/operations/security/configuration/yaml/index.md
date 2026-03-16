---
title: "本地配置"
date: 0001-01-01
summary: "本地配置（YAML） #  config/security/ 目录下包含 Easysearch 安全模块的本地 YAML 配置文件。这些文件在 bin/initialize.sh 初始化时自动加载到安全索引中，也可以手动编辑后重新加载。
通过本地 YAML 配置文件可以管理默认的内置用户或 隐藏的保留资源，例如 admin 管理员用户。除内置资源外，通过 INFINI Console 或 REST API 来创建其他用户、角色、映射和权限组通常更方便。
相关指南（先读这些） #    安全与多租户最佳实践  权限控制总览   配置文件概览 #     文件 用途 编辑方式     user.yml 内置用户定义 手动编辑或 API   role.yml 角色定义 手动编辑或 API   role_mapping.yml 角色映射 手动编辑或 API   privilege.yml 权限组（Action Group） 手动编辑或 API   config."
---


# 本地配置（YAML）

`config/security/` 目录下包含 Easysearch 安全模块的本地 YAML 配置文件。这些文件在 `bin/initialize.sh` 初始化时自动加载到安全索引中，也可以手动编辑后重新加载。

通过本地 YAML 配置文件可以管理默认的内置用户或[隐藏的保留资源](../access-control/api.md#隐藏的保留资源)，例如 `admin` 管理员用户。除内置资源外，通过 INFINI Console 或 REST API 来创建其他用户、角色、映射和权限组通常更方便。

## 相关指南（先读这些）

- [安全与多租户最佳实践]({{< relref "/docs/best-practices/security-and-multi-tenancy.md" >}})
- [权限控制总览]({{< relref "/docs/operations/security/access-control/_index.md" >}})

---

## 配置文件概览

| 文件 | 用途 | 编辑方式 |
|------|------|----------|
| `user.yml` | 内置用户定义 | 手动编辑或 API |
| `role.yml` | 角色定义 | 手动编辑或 API |
| `role_mapping.yml` | 角色映射 | 手动编辑或 API |
| `privilege.yml` | 权限组（Action Group） | 手动编辑或 API |
| `config.yml` | 认证后端配置 | 需重新加载生效 |
| `nodes_dn.yml` | 节点 DN 白名单 | 手动编辑或 API |
| `whitelist.yml` | API 白名单 | 手动编辑或 API |
| `audit.yml` | 审计日志配置 | 手动编辑或 API |

---

## user.yml

此文件是内置用户配置文件，用于定义系统的初始用户账户及其相关配置（如密码哈希、角色分配等）。

`admin` 用户是 Easysearch 系统中的默认超级管理员用户。

配置文件里面的密码不能是明文，必须使用 Hash 之后的密码，通过命令 `./bin/hash_password.sh -p <new-password>` 可以生成一个密码哈希。

### 默认配置

```yaml
---
_meta:
  type: "user"
  config_version: 2

admin:
  hash: "$2y$12$GNHStWSkQzhP9LnUbCVv2O2aFx67vXcFHcQvqC.vaE4AR66xAk0zG"
  reserved: true
  external_roles:
    - "admin"
  description: "Default admin user"
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `hash` | bcrypt 密码哈希（使用 `bin/hash_password.sh` 生成） |
| `reserved` | 是否为内置用户（`true` 不可通过 API 删除） |
| `hidden` | 是否在用户列表中隐藏 |
| `external_roles` | 外部角色列表（用于角色映射） |
| `attributes` | 自定义用户属性（用于细粒度权限控制） |
| `description` | 用户描述信息 |
| `static` | 是否为静态用户（`true` 仅通过文件管理） |

### 修改密码

使用密码哈希工具生成新的 bcrypt 哈希：

```bash
# 生成密码哈希
$ES_HOME/bin/hash_password.sh -p newpassword
```

或使用 API 修改（需要已认证）：

```bash
# 修改用户密码
curl -k -u admin:oldpassword -X PUT "https://localhost:9200/_security/account" \
  -H "Content-Type: application/json" \
  -d '{"password": "newpassword"}'
```

---

## role.yml

定义不同用户角色的权限。`replication_leader` 和 `replication_follower` 用于跨集群复制，`security` 用于允许通过 REST API 管理安全设置。

> ⚠️ **警告！**
> 以下角色已被弃用，将在未来的版本中删除，请勿使用：
> - `security_rest_api_access`
> - `cross_cluster_replication_leader_full_access`
> - `cross_cluster_replication_follower_full_access`

### 默认配置

```yaml
---
_meta:
  type: "role"
  config_version: 2

replication_leader:
  reserved: true
  description: "Grants read access to leader indices for cross-cluster replication."
  indices:
    - names:
        - "*"
      privileges:
        - "indices:admin/plugins/replication/index/setup/validate"
        - "indices:data/read/plugins/replication/changes"
        - "indices:data/read/plugins/replication/file_chunk"

replication_follower:
  reserved: true
  description: "Grants manage replication permissions on follower indices."
  cluster:
    - "cluster:admin/plugins/replication/autofollow/update"
  indices:
    - names:
        - "*"
      privileges:
        - "indices:admin/plugins/replication/index/setup/validate"
        - "indices:data/write/plugins/replication/changes"
        - "indices:admin/plugins/replication/index/start"
        - "indices:admin/plugins/replication/index/pause"
        - "indices:admin/plugins/replication/index/resume"
        - "indices:admin/plugins/replication/index/stop"
        - "indices:admin/plugins/replication/index/update"
        - "indices:admin/plugins/replication/index/status_check"

security:
  reserved: true
  description: "Grants access to the security REST API."
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `cluster` | 集群级权限列表（如 `cluster:monitor/*`、`cluster:admin/*` 等） |
| `indices` | 索引级权限列表，每项包含 `names`（索引模式）和 `privileges`（允许的操作） |
| `reserved` | 是否为内置角色 |
| `hidden` | 是否在角色列表中隐藏 |
| `description` | 角色描述信息 |
| `static` | 是否为静态角色（仅通过文件管理） |

### 索引权限项字段

| 字段 | 说明 |
|------|------|
| `names` | 索引名称模式列表，支持通配符 `*` |
| `privileges` | 允许的操作列表，可以是具体权限或权限集合名称 |
| `query` | 文档级安全（DLS），限制可见文档的查询条件 |
| `field_security` | 字段级安全（FLS），限制可见的字段列表 |
| `field_mask` | 字段脱敏，对指定字段进行脱敏处理 |

### 自定义角色示例

```yaml
logs_reader:
  reserved: false
  description: "Custom role for reading log indices."
  cluster:
    - "cluster:monitor/health"
  indices:
    - names:
        - "logs-*"
      privileges:
        - "read"
        - "search"
```

### 常见权限

**集群权限**：
- `cluster:*` — 所有集群操作
- `cluster:monitor/*` — 监控操作
- `cluster:admin/*` — 管理操作

**索引权限**：
- `indices:admin/*` — 索引管理
- `indices:data/write/*` — 写入操作
- `indices:data/read/*` — 读取操作

**权限集合名称**（在 `privileges` 中可直接使用）：
- `read` — 所有读取操作
- `write` — 所有写入操作
- `search` — 搜索操作
- `get` — 获取单个文档
- `index` — 索引文档
- `crud` — 增删改查组合
- `manage` — 索引管理
- `indices_all` — 所有索引操作

完整权限列表参见 [权限列表]({{< relref "/docs/operations/security/access-control/permissions.md" >}})。

---

## role_mapping.yml

将用户（users）、外部组/角色（external_roles）以及主机（hosts）映射到 Easysearch 的安全角色。

各角色的权限定义在 `role.yml`，此文件仅决定"谁拥有哪些角色"。每个角色条目下，只要当前主体命中 `users`、`external_roles` 或 `hosts` 中的任意一项，即会获得该角色。

> **提示**
> 示例文件中包含一条已弃用的映射（`all_access`）。请勿再使用此条目，未来版本会移除。

### 默认配置

```yaml
---
_meta:
  type: "role_mapping"
  config_version: 2

superuser:
  reserved: false
  external_roles:
    - "admin"
  description: "Maps admin to superuser"
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `external_roles` | 外部角色名称列表（LDAP 组、SAML 角色等） |
| `users` | 直接映射的用户名列表 |
| `and_external_roles` | 多个外部角色的 AND 条件（必须同时属于所有列出的角色） |
| `hosts` | 客户端主机名/IP 列表 |
| `description` | 映射描述信息 |

### 使用场景

- **LDAP**：映射 LDAP 组的 DN 到 Easysearch 角色
- **SAML**：映射 SAML 属性中的角色值
- **本地用户**：将 `user.yml` 中用户的 `external_roles` 映射到角色

### 自定义映射示例

```yaml
data_team:
  reserved: false
  external_roles:
    - "cn=data-team,ou=groups,dc=example,dc=com"
  users:
    - "analyst@example.com"
  description: "Maps data team LDAP group and specific users to data_team role"
```

---

## privilege.yml

定义自定义权限组（Action Group），在角色中复用。系统已内置多个常用权限组（如 `read`、`write`、`search` 等），此文件用于定义额外的自定义权限组。

### 默认配置

除了元数据之外，该文件默认为空，因为安全模块已内置了常用权限集合：

```yaml
---
_meta:
  type: "privilege"
  config_version: 2
```

### 自定义权限组示例

```yaml
SEARCH_ROLE:
  reserved: false
  allowed_actions:
    - "indices:data/read/search*"
    - "indices:data/read/get*"

WRITE_ROLE:
  reserved: false
  allowed_actions:
    - "indices:data/write/index*"
    - "indices:data/write/update*"
```

### 在角色中使用

在 `role.yml` 的 `privileges` 中引用权限组名称：

```yaml
logs_writer:
  indices:
    - names:
        - "logs-*"
      privileges:
        - "SEARCH_ROLE"
        - "WRITE_ROLE"
```

---

## config.yml

配置认证方式（Basic Auth、LDAP、SAML 等）和授权后端。详细说明请参见 [认证后端配置]({{< relref "./backend.md" >}})。

### 默认配置

```yaml
---
_meta:
  type: "config"
  config_version: 2

config:
  dynamic:
    http:
      anonymous_auth_enabled: false
      # xff:
      #   enabled: false
      #   internalProxies: '192\.168\.0\.10|192\.168\.0\.11' # regex pattern

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

    authz:
      basic_authz:
        http_enabled: true
        transport_enabled: true
        authorization_backend:
          type: noop
```

### 关键部分

| 配置块 | 说明 |
|--------|------|
| `http` | HTTP 层设置，包括匿名认证和 XFF 代理配置 |
| `authc` | 认证域配置，支持 `basic`、`proxy`、`kerberos`、`clientcert`、`jwt` 等类型 |
| `authz` | 授权后端配置，支持 `noop`、`ldap` 等类型 |

---

## nodes_dn.yml

定义允许加入集群的节点证书 DN（Distinguished Name）列表，与 `easysearch.yml` 中的 `security.nodes_dn` 作用相同，用于节点间 TLS 认证。

### 默认配置

```yaml
---
_meta:
  type: "nodesdn"
  config_version: 2

# Define nodesdn mapping name and corresponding values
# cluster1:
#   nodes_dn:
#     - CN=*.example.com
```

默认文件为空，表示不做额外的节点 DN 限制。

---

## whitelist.yml

限制非超级管理员用户可以访问的 API 端点。

### 默认配置

```yaml
---
_meta:
  type: "whitelist"
  config_version: 2

config:
  enabled: false
  requests:
    /_cluster/settings:
      - GET
    /_cat/nodes:
      - GET
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `enabled` | 是否启用白名单功能。`false` 表示关闭白名单，所有 API 均可访问 |
| `requests` | 白名单规则，键为 API 路径，值为允许的 HTTP 方法列表 |

> **注意**：白名单仅对非超级管理员用户生效。超级管理员（通过 admin 证书认证）始终可以访问所有 API。

---

## audit.yml

配置安全审计日志的记录范围和行为。

### 默认配置

```yaml
---
_meta:
  type: "audit"
  config_version: 2

config:
  enabled: true
  audit:
    enable_rest: true
    disabled_rest_categories:
      - AUTHENTICATED
      - GRANTED_PRIVILEGES
    enable_transport: true
    disabled_transport_categories:
      - AUTHENTICATED
      - GRANTED_PRIVILEGES
    ignore_users: []
    ignore_requests: []
    resolve_bulk_requests: false
    log_request_body: true
    resolve_indices: true
    exclude_sensitive_headers: true
  compliance:
    enabled: true
    internal_config: true
    external_config: false
    read_metadata_only: true
    read_watched_fields: {}
    read_ignore_users: []
    write_metadata_only: true
    write_log_diffs: false
    write_watched_indices: []
    write_ignore_users: []
```

### 审计配置字段

| 字段 | 说明 |
|------|------|
| `enabled` | 是否启用审计日志 |
| `enable_rest` | 是否审计 REST API 请求 |
| `enable_transport` | 是否审计传输层请求 |
| `disabled_rest_categories` | REST 层排除的审计类别 |
| `disabled_transport_categories` | 传输层排除的审计类别 |
| `include_users` | 仅审计指定用户列表（可选，支持通配符） |
| `ignore_users` | 排除审计的用户列表（支持通配符） |
| `ignore_requests` | 排除审计的请求列表（支持通配符） |
| `resolve_bulk_requests` | 是否记录 bulk 请求中的每个操作 |
| `log_request_body` | 是否记录请求体 |
| `resolve_indices` | 是否解析请求涉及的索引（含别名与通配符） |
| `exclude_sensitive_headers` | 是否排除敏感 HTTP 头（如 Authorization） |

### 合规性配置字段

| 字段 | 说明 |
|------|------|
| `enabled` | 是否启用合规审计 |
| `internal_config` | 是否记录安全配置的内部变更 |
| `external_config` | 是否记录外部配置文件的变更 |
| `read_metadata_only` | 读取事件是否仅记录元数据 |
| `read_watched_fields` | 监控读取的索引和字段 |
| `read_ignore_users` | 读取审计忽略用户列表 |
| `write_metadata_only` | 写入事件是否仅记录元数据 |
| `write_log_diffs` | 是否记录文档更新的差异 |
| `write_watched_indices` | 监控写入的索引列表 |
| `write_ignore_users` | 写入审计忽略用户列表 |

---

## 修改配置文件

### 方式 1：手动编辑（需重新加载）

修改 `config/security/` 目录下的 YAML 文件后，需要重新初始化安全索引使配置生效：

```bash
# 编辑配置
vi config/security/config.yml

# 重新加载安全配置（以管理员身份删除安全索引后重启）
curl -XDELETE -k --cert admin.crt --key admin.key 'https://localhost:9200/.security'
# 然后重启 Easysearch 服务
```

也可以使用初始化脚本：

```bash
bin/initialize.sh
```

### 方式 2：使用 API（即时生效）

大多数配置可以通过安全 REST API 修改，即时生效：

```bash
# 获取现有角色
curl -k -u admin:password "https://localhost:9200/_security/role/my_role"

# 创建或修改角色
curl -k -u admin:password -X PUT "https://localhost:9200/_security/role/my_role" \
  -H "Content-Type: application/json" \
  -d '{
    "cluster": ["cluster:monitor/health"],
    "indices": [{
      "names": ["logs-*"],
      "privileges": ["read", "search"]
    }]
  }'

# 创建用户
curl -k -u admin:password -X PUT "https://localhost:9200/_security/user/newuser" \
  -H "Content-Type: application/json" \
  -d '{"password": "StrongPass123!", "external_roles": ["admin"]}'
```

> **注意**
> 任何对本地配置文件的修改（如 `user.yml`、`role.yml`、`role_mapping.yml`），修改后必须以管理员身份删除 `.security` 索引，然后重启服务才能生效：
> ```bash
> curl -XDELETE -k --cert admin.crt --key admin.key 'https://localhost:9200/.security'
> ```

---

## 文件权限

安全配置文件包含敏感信息（如密码哈希），应该限制访问权限：

```bash
# 只有 Easysearch 进程用户可以读取
chmod 600 config/security/*.yml

# 配置目录权限
chmod 700 config/security/
```

