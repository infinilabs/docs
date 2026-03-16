---
title: "API 接口"
date: 0001-01-01
summary: "API #  通过 REST API 可以管理用户、角色、角色映射、权限集合和租户。
API 的访问控制 #  您可以控制哪些角色可以访问安全相关的 API，在配置文件 easysearch.yml:
security.restapi.roles_enabled: [&#34;&lt;role&gt;&#34;, ...] 如果希望阻止访问特定的 API：
security.restapi.endpoints_disabled.&lt;role&gt;.&lt;endpoint&gt;: [&#34;&lt;method&gt;&#34;, ...] 参数 endpoint 可以是:
 PRIVILEGE ROLE ROLE_MAPPING USER CONFIG CACHE  参数 method 可以是:
 GET PUT POST DELETE PATCH  例如，以下配置授予三个角色对 REST API 的访问权限，但随后会阻止 test-role 发送 PUT, POST, DELETE, 或 PATCH 到 _security/role 或 _security/user :
security.restapi.roles_enabled: [&#34;superuser&#34;, &#34;security&#34;, &#34;test-role&#34;] security.restapi.endpoints_disabled.test-role.ROLE: [&#34;PUT&#34;, &#34;POST&#34;, &#34;DELETE&#34;, &#34;PATCH&#34;] security.restapi.endpoints_disabled.test-role.USER: [&#34;PUT&#34;, &#34;POST&#34;, &#34;DELETE&#34;, &#34;PATCH&#34;] 要为 API 配置 使用 PUT 和 PATCH 方法，请将以下行添加到 easysearch."
---


# API

通过 REST API 可以管理用户、角色、角色映射、权限集合和租户。

## API 的访问控制

您可以控制哪些角色可以访问安全相关的 API，在配置文件 `easysearch.yml`:

```yml
security.restapi.roles_enabled: ["<role>", ...]
```

如果希望阻止访问特定的 API：

```yml
security.restapi.endpoints_disabled.<role>.<endpoint>: ["<method>", ...]
```

参数 `endpoint` 可以是:

- PRIVILEGE
- ROLE
- ROLE_MAPPING
- USER
- CONFIG
- CACHE

参数 `method` 可以是:

- GET
- PUT
- POST
- DELETE
- PATCH

例如，以下配置授予三个角色对 REST API 的访问权限，但随后会阻止 `test-role` 发送 PUT, POST, DELETE, 或 PATCH 到 `_security/role` 或 `_security/user` :

```yml
security.restapi.roles_enabled:
  ["superuser", "security", "test-role"]
security.restapi.endpoints_disabled.test-role.ROLE:
  ["PUT", "POST", "DELETE", "PATCH"]
security.restapi.endpoints_disabled.test-role.USER:
  ["PUT", "POST", "DELETE", "PATCH"]
```

要为 [API 配置](api.md#api) 使用 PUT 和 PATCH 方法，请将以下行添加到 `easysearch.yml`：

```yml
security.unsupported.restapi.allow_securityconfig_modification: true
```

## 隐藏的保留资源

通过修改以下参数，您可以将用户、角色、角色映射和权限集合标记为保留。随后将无法使用 REST API 修改这些资源。

```yml
dashboard_user:
  reserved: true
```

同样，可以将用户、角色、角色映射和权限集合标记为隐藏。REST API 不会返回此标志设置为 true 的资源：

```yml
dashboard_user:
  hidden: true
```

隐藏的资源默认就是保留的资源。

## 账号信息

### 获取账号信息

返回当前用户的帐户详细信息。例如，如果您是以 `admin` 用户身份对请求进行登录的，则响应将包含该用户的详细信息。

#### 请求

```
GET _security/account
```

#### 示例

```json
curl -XGET -k -u booksuser:password 'https://localhost:9200/_security/account?pretty'
{
  "username" : "booksuser",
  "reserved" : false,
  "hidden" : false,
  "builtin" : true,
  "external_roles" : [ ],
  "attributes" : [ ],
  "roles" : [
    "booksrole"
  ],
  "cluster_privileges" : [ ],
  "index_privileges" : [
    {
      "role" : "booksrole",
      "names" : [
        "books"
      ],
      "privileges" : [
        "read"
      ]
    }
  ]
}
```

字段说明：

- `cluster_privileges`：聚合后的集群级权限（多角色去重合并；即使包含 `*` 也会保留其他具体项）。
- `index_privileges`：索引级权限明细，包含来源角色、索引模式与权限列表。

### 修改密码

修改当前用户的密码。

#### 请求

```json
PUT _security/account
{
    "current_password" : "old-password",
    "password" : "new-password"
}
```

#### 示例

```json
✗ curl-json -XPUT -k -u booksuser:password 'https://localhost:9200/_security/account' -d'{
    "current_password" : "password",
    "password" : "admin1"
}'

{"status":"OK","message":"'booksuser' updated."}%
```

---

## 权限集合

### 获取权限集合

获取一个权限集合信息。

#### 请求

```
GET _security/privilege/<name>
```

#### 示例

```json
{
  "custom_action_group": {
    "reserved": false,
    "hidden": false,
    "privileges": [
      "dashboard_all_read",
      "indices:admin/aliases/get",
      "indices:admin/aliases/exists"
    ],
    "description": "My custom action group",
    "static": false
  }
}
```

### 获取权限集合列表

获取所有的权限集合列表。

#### 请求

```
GET _security/privilege/
```

#### 示例

```json
{
  "read": {
    "reserved": true,
    "hidden": false,
    "privileges": [
      "indices:data/read*",
      "indices:admin/mappings/fields/get*"
    ],
    "type": "index",
    "description": "Allow all read operations",
    "static": true
  },
  ...
}
```

### 删除一个权限集合

#### 请求

```
DELETE _security/privilege/<name>
```

#### 示例

```json
{
  "status": "OK",
  "message": "privilege SEARCH deleted."
}
```

### 创建一个权限集合

创建或替换指定的限集合。

#### 请求

```json
PUT _security/privilege/<name>
{
  "privileges": [
    "indices:data/write/index*",
    "indices:data/write/update*",
    "indices:admin/mapping/put",
    "indices:data/write/bulk*",
    "read",
    "write"
  ]
}
```

#### 示例

```json
{
  "status": "CREATED",
  "message": "'my-action-group' created."
}
```

### 修改权限集合

修改权限集合的属性。

#### 请求

```json
PATCH _security/privilege/<name>
[
  {
    "op": "replace", "path": "/privileges", "value": ["indices:admin/create", "indices:admin/mapping/put"]
  }
]
```

#### 示例

```json
{
  "status": "OK",
  "message": "privilege SEARCH deleted."
}
```

### 修改一批权限集合

一次修改多个权限集合。

#### 请求

```json
PATCH _security/privilege
[
  {
    "op": "add", "path": "/CREATE_INDEX", "value": { "privileges": ["indices:admin/create", "indices:admin/mapping/put"] }
  },
  {
    "op": "remove", "path": "/CRUD"
  }
]
```

#### 示例

```json
{
  "status": "OK",
  "message": "privilege SEARCH deleted."
}
```

---

## 用户

这些 API 允许您创建、更新和删除内部用户。

### 获取用户

#### 请求

```
GET _security/user/<username>
```

#### 实例

```json
{
  "admin": {
    "hash": "",
    "roles": ["captains", "starfleet"],
    "attributes": {
      "attribute1": "value1",
      "attribute2": "value2"
    }
  }
}
```

### 获取用户列表

#### 请求

```
GET _security/user/
```

#### 示例

```json
{
  "admin": {
    "hash": "",
    "roles": ["captains", "starfleet"],
    "attributes": {
      "attribute1": "value1",
      "attribute2": "value2"
    }
  }
}
```

### 删除用户

#### 请求

```
DELETE _security/user/<username>
```

#### 示例

```json
{
  "status": "OK",
  "message": "user admin deleted."
}
```

### 创建用户

创建或替换指定的用户。您必须指定 `password` (明文)或 `hash` (哈希后的密码)。如果您指定 `password`,安全模块会在存储密码之前自动对密码进行哈希处理。

请注意，您在 `roles` 数组中提供的任何角色都必须已经存在，安全模块才能将用户映射到该角色。要查看预定义的角色，请参阅[这里](api.md#获取角色列表)。有关如何创建角色的说明，请参阅 [创建角色](api.md#创建一个角色)。

#### 请求

```json
PUT _security/user/<username>
{
  "password": "adminpass",
  "roles": ["maintenance_staff", "weapons"],
  "external_roles": ["captains", "starfleet"],
  "attributes": {
    "attribute1": "value1",
    "attribute2": "value2"
  }
}
```

#### 示例

```json
{
  "status": "CREATED",
  "message": "User admin created"
}
```

### 修改用户

修改用户信息。

#### Request

```json
PATCH _security/user/<username>
[
  {
    "op": "replace", "path": "/external_roles", "value": ["klingons"]
  },
  {
    "op": "replace", "path": "/roles", "value": ["ship_manager"]
  },
  {
    "op": "replace", "path": "/attributes", "value": { "newattribute": "newvalue" }
  }
]
```

#### 示例

```json
{
  "status": "OK",
  "message": "'admin' updated."
}
```

### 修改一批用户

一次修改一批用户的操作。

#### 请求

```json
PATCH _security/user
[
  {
    "op": "add", "path": "/spock", "value": { "password": "testpassword1", "external_roles": ["testrole1"] }
  },
  {
    "op": "add", "path": "/worf", "value": { "password": "testpassword2", "external_roles": ["testrole2"] }
  },
  {
    "op": "remove", "path": "/riker"
  }
]
```

#### 示例

```json
{
  "status": "OK",
  "message": "Resource updated."
}
```

---

## 角色

### 获取角色

获取一个角色信息。

#### 请求

```
GET _security/role/<role>
```

#### 示例

```json
{
  "test-role": {
    "reserved": false,
    "hidden": false,
    "cluster": ["cluster_composite_ops", "indices_monitor"],
    "indices": [
      {
        "names": ["movies*"],
        "query": "",
        "field_security": [],
        "field_mask": [],
        "privileges": ["read"]
      }
    ],
    "static": false
  }
}
```

### 获取角色列表

#### 请求

```
GET _security/role/
```

#### 示例

```json
{
  "manage_snapshots": {
    "reserved": true,
    "hidden": false,
    "description": "Provide the minimum permissions for managing snapshots",
    "cluster": [
      "manage_snapshots"
    ],
    "indices": [{
      "names": [
        "*"
      ],
      "field_security": [],
      "field_mask": [],
      "privileges": [
        "indices:data/write/index",
        "indices:admin/create"
      ]
    }],
    "static": true
  },
  ...
}
```

### 删除一个角色

#### 请求

```
DELETE _security/role/<role>
```

#### 示例

```json
{
  "status": "OK",
  "message": "role test-role deleted."
}
```

### 创建一个角色

#### 请求

```json
PUT _security/role/<role>
{
  "cluster": [
    "cluster_composite_ops",
    "indices_monitor"
  ],
  "indices": [{
    "names": [
      "movies*"
    ],
    "query": "",
    "field_security": [],
    "field_mask": [],
    "privileges": [
      "read"
    ]
  }]
}
```

#### 示例

```json
{
  "status": "OK",
  "message": "'test-role' updated."
}
```

### 修改角色

修改角色的属性。

#### 请求

```json
PATCH _security/role/<role>
[
  {
    "op": "replace", "path": "/indices/0/field_security", "value": ["myfield1", "myfield2"]
  },
  {
    "op": "remove", "path": "/indices/0/query"
  }
]
```

#### 示例

```json
{
  "status": "OK",
  "message": "'<role>' updated."
}
```

### 批量修改角色

一次操作批量修改角色信息。

#### 请求

```json
PATCH _security/role
[
  {
    "op": "replace", "path": "/role1/indices/0/field_security", "value": ["test1", "test2"]
  },
  {
    "op": "remove", "path": "/role1/indices/0/query"
  },
  {
    "op": "add", "path": "/role2/cluster", "value": ["manage_snapshots"]
  }
]
```

#### 示例

```json
{
  "status": "OK",
  "message": "Resource updated."
}
```

---

## 角色映射

### 获取一个角色映射

获取一个角色映射信息。

#### 请求

```
GET _security/role_mapping/<role>
```

#### 示例

```json
{
  "role_starfleet": {
    "external_roles": [
      "starfleet",
      "captains",
      "defectors",
      "cn=ldaprole,ou=groups,dc=example,dc=com"
    ],
    "hosts": ["*.starfleetintranet.com"],
    "users": ["worf"]
  }
}
```

### 获取角色映射列表。

获取所有的角色映射列表。

#### 请求

```
GET _security/role_mapping
```

#### 示例

```json
{
  "role_starfleet": {
    "external_roles": [
      "starfleet",
      "captains",
      "defectors",
      "cn=ldaprole,ou=groups,dc=example,dc=com"
    ],
    "hosts": ["*.starfleetintranet.com"],
    "users": ["worf"]
  }
}
```

### 删除角色映射

#### 请求

```
DELETE _security/role_mapping/<role>
```

#### 示例

```json
{
  "status": "OK",
  "message": "'my-role' deleted."
}
```

### 创建角色映射

#### 请求

```json
PUT _security/role_mapping/<role>
{
  "external_roles" : [ "starfleet", "captains", "defectors", "cn=ldaprole,ou=groups,dc=example,dc=com" ],
  "hosts" : [ "*.starfleetintranet.com" ],
  "users" : [ "worf" ]
}
```

#### 示例

```json
{
  "status": "CREATED",
  "message": "'my-role' created."
}
```

### 修改角色映射

#### 请求

```json
PATCH _security/role_mapping/<role>
[
  {
    "op": "replace", "path": "/users", "value": ["myuser"]
  },
  {
    "op": "replace", "path": "/external_roles", "value": ["mybackendrole"]
  }
]
```

#### 示例

```json
{
  "status": "OK",
  "message": "'my-role' updated."
}
```

### 批量修改角色映射

#### 请求

```json
PATCH _security/role_mapping
[
  {
    "op": "add", "path": "/human_resources", "value": { "users": ["user1"], "external_roles": ["backendrole2"] }
  },
  {
    "op": "add", "path": "/finance", "value": { "users": ["user2"], "external_roles": ["backendrole2"] }
  }
]
```

#### 示例

```json
{
  "status": "OK",
  "message": "Resource updated."
}
```

## 缓存

### 刷新缓存

#### 请求

```
DELETE _security/cache
```

#### 示例

```json
{
  "status": "OK",
  "message": "Cache flushed successfully."
}
```

## admin

### 修改admin密码

进入Easysearch的config路径下，通过证书的方式进行密码修改，具体可查看[这里](https://infinilabs.cn/blog/2025/easysearch-reset-admin-password/)。

#### 示例

```bash
cd easysearch/config
#执行 admin 密码修改
curl -k -XPUT --cert admin.crt --key admin.key -H 'Content-Type: application/json' 'https://localhost:9200/_security/user/admin' -d '
{
  "password": "C0mp1exP@easysearch",
  "external_roles": ["admin"]
}'

#用新密码验证是否修改成功
curl -ku 'admin:C0mp1exP@easysearch' https://localhost:9200
```

