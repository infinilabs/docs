---
title: "Access Token"
date: 0001-01-01
summary: "Access Token #  Access Token 提供了一种面向程序调用的认证方式。客户端可以先由管理员签发 token，后续通过 HTTP 头 X-API-TOKEN 访问 Easysearch API，而不需要在每次请求中传递用户名和密码。
这一能力适合以下场景：
 服务到服务的 API 调用 临时授予只读或特定动作权限 为自动化脚本、采集任务、运维工具发放独立凭据  相关指南（先读这些） #    权限控制总览  用户与角色  权限列表  安全 API  使用前提 #  在使用 Access Token 之前，请先确认：
 安全模块已经启用。 你可以使用 superadmin 身份或 superuser 角色访问安全 API。 调用方已经明确需要的权限范围。   当前实现中，创建 Access Token 需要 superadmin 身份，或拥有 superuser 角色。在默认发行配置下，初始化后的 admin 用户通常会映射到 superuser；如果你修改过默认密码或角色映射，请改用实际具备相应权限的账户。
 接口概览 #     操作 方法 路径     创建 token POST /_security/access_token   更新 token PUT /_security/access_token/{token_id}   删除 token DELETE /_security/access_token/{token_id}   搜索 token GET /_security/access_token/search    创建 Access Token #  创建请求至少需要指定："
---


# Access Token

Access Token 提供了一种面向程序调用的认证方式。客户端可以先由管理员签发 token，后续通过 HTTP 头 `X-API-TOKEN` 访问 Easysearch API，而不需要在每次请求中传递用户名和密码。

这一能力适合以下场景：

- 服务到服务的 API 调用
- 临时授予只读或特定动作权限
- 为自动化脚本、采集任务、运维工具发放独立凭据

## 相关指南（先读这些）

- [权限控制总览]({{< relref "/docs/operations/security/access-control/_index.md" >}})
- [用户与角色]({{< relref "/docs/operations/security/access-control/users-roles.md" >}})
- [权限列表]({{< relref "/docs/operations/security/access-control/permissions.md" >}})
- [安全 API]({{< relref "/docs/operations/security/access-control/api.md" >}})

## 使用前提

在使用 Access Token 之前，请先确认：

1. 安全模块已经启用。
2. 你可以使用 superadmin 身份或 `superuser` 角色访问安全 API。
3. 调用方已经明确需要的权限范围。

> 当前实现中，创建 Access Token 需要 superadmin 身份，或拥有 `superuser` 角色。在默认发行配置下，初始化后的 `admin` 用户通常会映射到 `superuser`；如果你修改过默认密码或角色映射，请改用实际具备相应权限的账户。

## 接口概览

| 操作 | 方法 | 路径 |
|------|------|------|
| 创建 token | `POST` | `/_security/access_token` |
| 更新 token | `PUT` | `/_security/access_token/{token_id}` |
| 删除 token | `DELETE` | `/_security/access_token/{token_id}` |
| 搜索 token | `GET` | `/_security/access_token/search` |

## 创建 Access Token

创建请求至少需要指定：

- `name`：token 名称
- `permissions`：权限列表
- `expire_in`：过期时间，Unix 秒级时间戳，且必须大于当前时间

示例：

> 下文里的 `admin:admin` 仅用于说明默认发行配置下的调用方式；如果你的环境已经修改默认密码，或者 `admin` 不再映射到 `superuser`，请替换为实际具备 superadmin 或 `superuser` 权限的账户。

```bash
EXPIRE_IN=$(($(date +%s) + 3600))

curl -k -u admin:admin -X POST "https://localhost:9200/_security/access_token" \
  -H 'Content-Type: application/json' \
  -d "{
    \"name\": \"backup-bot\",
    \"description\": \"token for snapshot inspection\",
    \"permissions\": [
      \"cluster_monitor\",
      \"indices:data/read/*\"
    ],
    \"expire_in\": ${EXPIRE_IN}
  }"
```

响应示例：

```json
{
  "access_token": "a3f72d6f-2f9a-4a4d-89a0-4f261ee0d2adPq7fG8mT2kR9xN4vH1cD6uY3bL5sQ0wE8zA2jK4nM7pR1tV9yB6cF3dS0hJ2gL5",
  "expire_in": 1772812800
}
```

说明：

1. 返回结果只包含 `access_token` 和 `expire_in`。
2. 通用 Access Token 的字符串格式为 `<uuid><64_random_chars>`。
3. token 文档会保存在 `.security` 索引中。

## 权限字段怎么写

`permissions` 支持两类值：

1. 已定义的 privilege 名，例如：
   - `read`
   - `write`
   - `cluster_monitor`
   - `cluster_composite_ops_ro`
2. 原始 action pattern，例如：
   - `indices:data/read/search`
   - `indices:data/read/*`
   - `cluster:monitor/*`

示例：

```json
{
  "permissions": [
    "cluster_monitor",
    "indices:data/read/*"
  ]
}
```

实践建议：

- 优先使用已有 privilege 名，便于统一治理。
- 只在确实需要精确控制时，才直接写原始 action pattern。
- 不要直接给过宽的权限，例如无必要时不要使用全量通配符。

## 使用 Access Token 调用 API

拿到 token 后，请在请求头中传入：

```http
X-API-TOKEN: <access_token>
```

例如，使用只读 token 调用 `_cat/nodes`：

```bash
curl -k "https://localhost:9200/_cat/nodes?format=json" \
  -H "X-API-TOKEN: <access_token>"
```

再例如，使用搜索权限调用 `_search`：

```bash
curl -k "https://localhost:9200/logs-2026.04/_search" \
  -H "X-API-TOKEN: <access_token>" \
  -H 'Content-Type: application/json' \
  -d '{
    "query": {
      "match_all": {}
    }
  }'
```

认证与授权行为如下：

1. 服务端先根据 `X-API-TOKEN` 查找 token 文档。
2. 若 token 不存在、已过期或状态不是 `active`，请求返回 401。
3. 若 token 认证成功，但请求动作不在允许范围内，请求返回 403。

## 搜索已有 Token

可以通过名称过滤，也可以传 Query DSL 请求体。

按名称搜索：

```bash
curl -k -u admin:admin \
  "https://localhost:9200/_security/access_token/search?name=backup-bot"
```

使用请求体搜索：

```bash
curl -k -u admin:admin -X GET "https://localhost:9200/_security/access_token/search" \
  -H 'Content-Type: application/json' \
  -d '{
    "query": {
      "match": {
        "name": "backup"
      }
    },
    "sort": [
      { "created": "desc" }
    ],
    "size": 10
  }'
```

说明：

1. 默认按 `created desc` 排序。
2. 默认 `size=10`。
3. `size` 最大为 `1000`。

## 更新 Token

更新接口路径中的 `{token_id}` 取自 token 字符串前 36 位 UUID，或者搜索结果中的文档 `_id`。

示例：

```bash
curl -k -u admin:admin -X PUT "https://localhost:9200/_security/access_token/a3f72d6f-2f9a-4a4d-89a0-4f261ee0d2ad" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "backup-bot-v2",
    "description": "updated token",
    "permissions": [
      "cluster_monitor",
      "indices:data/read/*"
    ],
    "expire_in": 1772899200
  }'
```

响应示例：

```json
{
  "_id": "a3f72d6f-2f9a-4a4d-89a0-4f261ee0d2ad",
  "result": "updated"
}
```

注意：

1. 当前实现里，`name` 在更新时仍然必填。
2. `description`、`permissions`、`expire_in` 可以按需更新。
3. 不能通过该接口修改 `access_token` 字符串本身。

## 删除 Token

```bash
curl -k -u admin:admin -X DELETE \
  "https://localhost:9200/_security/access_token/a3f72d6f-2f9a-4a4d-89a0-4f261ee0d2ad"
```

响应示例：

```json
{
  "_id": "a3f72d6f-2f9a-4a4d-89a0-4f261ee0d2ad",
  "result": "deleted"
}
```

## 常见错误

| HTTP | 场景 | 示例 |
|------|------|------|
| `400` | 请求体缺失或字段非法 | `name is required` |
| `403` | 既不是 superadmin，也没有 `superuser` 角色 | `only superadmin or superuser can create access token` |
| `404` | 更新或删除的 token 不存在 | `access token not found` |
| `401` | token 无效、过期或已失效 | `invalid access token` |
| `403` | token 已认证但无权执行请求动作 | `no permissions for ... and access token` |

## 与 enrollment token 的区别

`/_cluster/enroll/node` 也会返回一个 `access_token`，但它和这里的通用 Access Token 不是同一个使用场景。

区别如下：

1. 通用 Access Token 由 superadmin 或拥有 `superuser` 角色的用户通过 `/_security/access_token` 主动创建。
2. enrollment access token 由 `/_cluster/enroll/node` 内部签发。
3. enrollment access token 的默认 TTL 为 1 小时，但不是“签发后固定 1 小时到期”。
4. 同一个 enroll token 重复成功调用 `/_cluster/enroll/node` 时，会复用同一个 access token，并重新计算 `expire_in` 写回，也就是按成功调用时间做滑动续期。
5. enrollment access token 的权限固定为 `cluster_monitor`，主要用于节点加入集群时的只读引导请求。

因此：

- 如果你的目标是让脚本或应用访问业务 API，请使用通用 Access Token。
- 如果你的目标是节点 enrollment，请使用 `/_cluster/enroll/node` 返回的 token。

## 实践建议

1. 为不同调用方签发不同 token，不要多人或多服务共用一个 token。
2. `expire_in` 尽量设置较短，并定期轮换。
3. 优先授予最小权限集合。
4. 对高权限 token 保留审计与使用记录。
5. 如果只需要简单的管理操作，优先考虑通过角色体系管理用户，而不是长期使用高权限 token。

