---
title: "Access Token"
date: 0001-01-01
summary: "Access Token #  Access Token 提供了一种面向程序调用的认证方式。客户端可以先由管理员签发 token，后续通过 HTTP 头 X-API-TOKEN 访问 Easysearch API，而不需要在每次请求中传递用户名和密码。
这一能力适合以下场景：
 服务到服务的 API 调用 临时授予只读或特定动作权限 为自动化脚本、采集任务、运维工具发放独立凭据  相关指南（先读这些） #   权限控制总览 用户与角色 权限列表 安全 API  使用前提 #  在使用 Access Token 之前，请先确认：
 安全模块已经启用。 你可以使用 superadmin 身份或 superuser 角色访问安全 API。 调用方已经明确需要的权限范围。   当前实现中，创建 Access Token 需要 superadmin 身份，或拥有 superuser 角色。在默认发行配置下，初始化后的 admin 用户通常会映射到 superuser；如果你修改过默认密码或角色映射，请改用实际具备相应权限的账户。
 接口概览 #     操作 方法 路径     创建 token POST /_security/access_token   更新 token PUT /_security/access_token/{token_id}   删除 token DELETE /_security/access_token/{token_id}   搜索 token GET / POST /_security/access_token/search    创建 Access Token #  创建请求至少需要指定："
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
| 搜索 token | `GET` / `POST` | `/_security/access_token/search` |

## 创建 Access Token

创建请求至少需要指定：

- `name`：token 名称
- `cluster` 或 `indices`：权限描述，至少提供一种

其中 `expire_in` 为可选字段：

- 如果显式传入：必须是一个大于当前时间的 Unix 秒级时间戳
- 如果省略：服务端默认设置为“当前时间 + 3600 秒（1 小时）”

示例：

> 下文里的 `admin:admin` 仅用于说明默认发行配置下的调用方式；如果你的环境已经修改默认密码，或者 `admin` 不再映射到 `superuser`，请替换为实际具备 superadmin 或 `superuser` 权限的账户。

```bash
curl -k -u admin:admin -X POST "https://localhost:9200/_security/access_token" \
  -H 'Content-Type: application/json' \
  -d "{
    \"name\": \"backup-bot\",
    \"description\": \"token for snapshot inspection\",
    \"cluster\": [
      \"cluster_monitor\"
    ],
    \"indices\": [
      {
        \"names\": [
          \"snapshots*\"
        ],
        \"query\": \"\",
        \"field_security\": [],
        \"field_mask\": [],
        \"privileges\": [
          \"read\"
        ]
      }
    ]
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
3. token 文档会以 `cluster` / `indices` 权限描述形式保存在 `.security` 索引中。
4. 如果创建请求里省略 `expire_in`，响应中的 `expire_in` 会是服务端按默认 TTL（1 小时）计算后的到期时间。
5. `access_token` 只会在创建成功时返回一次；搜索、更新和删除接口都不会再次返回这个字段，请在创建后立即安全保存。

## 权限描述怎么写

Access Token 的权限结构与创建角色时保持一致，使用顶层 `cluster` 与 `indices` 两个字段。

约束如下：

1. `cluster`：字符串数组，表示集群级权限。
2. `indices`：对象数组，表示索引级权限。
3. `cluster` 与 `indices` 至少要提供一种，不能同时为空。
4. `indices` 中每个对象至少需要：
   - `names`：索引名或模式数组
   - `privileges`：权限数组
5. `indices` 中可选字段：
   - `query`：DLS 查询字符串
   - `field_security`：允许访问的字段列表
   - `field_mask`：字段脱敏列表

`cluster` 和 `indices[*].privileges` 都支持两类值：

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
  "cluster": [
    "cluster_monitor"
  ],
  "indices": [
    {
      "names": [
        "logs-*"
      ],
      "query": "",
      "field_security": [],
      "field_mask": [],
      "privileges": [
        "read"
      ]
    }
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

例如，使用具备集群监控权限的 token 调用 `_cat/nodes`：

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

如果你希望使用 token 执行常见的文档写入 API，例如：

- `PUT /<index>/_doc/<id>`
- `POST /<index>/_bulk`
- `POST /<index>/_update/<id>`
- `DELETE /<index>/_doc/<id>`

当前实现下，通常需要同时授予：

1. 集群级 `cluster_composite_ops`
2. 索引级 `write`、`index` 或更细粒度的写权限

例如：

```json
{
  "cluster": [
    "cluster_composite_ops"
  ],
  "indices": [
    {
      "names": [
        "logs-*"
      ],
      "query": "",
      "field_security": [],
      "field_mask": [],
      "privileges": [
        "write"
      ]
    }
  ]
}
```

如果只授予索引级 `write` / `index`，但没有授予 `cluster_composite_ops`，这些 REST 写入请求在当前实现中可能会因为 `indices:data/write/bulk` 权限不足而返回 403。

## 常用 API 权限配置示例

下面这些示例展示的是创建 token 时请求体里的 `cluster` / `indices` 权限配置片段，适用于当前实现里较常见的 API 组合。

| 场景 | 常见 API | 推荐权限 |
|------|----------|----------|
| 集群监控 | `GET /_cluster/health`、`GET /_cat/nodes?format=json` | `cluster_monitor` |
| 索引搜索与单文档读取 | `GET /<index>/_search`、`GET /<index>/_count`、`GET /<index>/_doc/<id>` | `read` |
| 批量读取 | `POST /_mget` | `cluster_composite_ops_ro` + `read` |
| 常见文档写入（不需要读） | `PUT /<index>/_doc/<id>`、`POST /<index>/_update/<id>`、`POST /<index>/_bulk`、`DELETE /<index>/_doc/<id>` | `cluster_composite_ops` + `write` |
| 常见文档 CRUD | 搜索、获取、`_mget`、索引、更新、bulk、删除 | `cluster_composite_ops` + `crud` |
| 查看索引元数据 | `GET /<index>/_settings`、`GET /<index>/_stats`、`GET /_cat/indices/<index>?format=json` | `indices_monitor`；若需要 `_cat/indices/<index>`，再加 `cluster_monitor` |
| 修改 mapping | `PUT /<index>/_mapping` | `manage` |

示例 1：只允许集群监控 API

```json
{
  "cluster": [
    "cluster_monitor"
  ],
  "indices": []
}
```

示例 2：允许搜索、`_count` 和 `GET /_doc/<id>`

```json
{
  "cluster": [],
  "indices": [
    {
      "names": [
        "logs-*"
      ],
      "query": "",
      "field_security": [],
      "field_mask": [],
      "privileges": [
        "read"
      ]
    }
  ]
}
```

示例 3：允许 `POST /_mget`

```json
{
  "cluster": [
    "cluster_composite_ops_ro"
  ],
  "indices": [
    {
      "names": [
        "logs-*"
      ],
      "query": "",
      "field_security": [],
      "field_mask": [],
      "privileges": [
        "read"
      ]
    }
  ]
}
```

说明：

1. 仅有索引级 `read` 时，`_search` 和 `GET /_doc/<id>` 可以正常使用。
2. `POST /_mget` 额外依赖 `cluster_composite_ops_ro`。

示例 4：允许常见文档写入，但不授予读权限

```json
{
  "cluster": [
    "cluster_composite_ops"
  ],
  "indices": [
    {
      "names": [
        "logs-*"
      ],
      "query": "",
      "field_security": [],
      "field_mask": [],
      "privileges": [
        "write"
      ]
    }
  ]
}
```

这个组合适用于：

- `PUT /<index>/_doc/<id>`
- `POST /<index>/_update/<id>`
- `POST /<index>/_bulk`
- `DELETE /<index>/_doc/<id>`

但不允许：

- `GET /<index>/_search`
- `GET /<index>/_doc/<id>`
- `POST /_mget`

示例 5：允许常见文档 CRUD

```json
{
  "cluster": [
    "cluster_composite_ops"
  ],
  "indices": [
    {
      "names": [
        "logs-*"
      ],
      "query": "",
      "field_security": [],
      "field_mask": [],
      "privileges": [
        "crud"
      ]
    }
  ]
}
```

这个组合通常可覆盖：

- `GET /<index>/_search`
- `GET /<index>/_count`
- `GET /<index>/_doc/<id>`
- `POST /_mget`
- `PUT /<index>/_doc/<id>`
- `POST /<index>/_update/<id>`
- `POST /<index>/_bulk`
- `DELETE /<index>/_doc/<id>`

示例 6：查看索引元数据与 cat 结果

```json
{
  "cluster": [
    "cluster_monitor"
  ],
  "indices": [
    {
      "names": [
        "logs-*"
      ],
      "query": "",
      "field_security": [],
      "field_mask": [],
      "privileges": [
        "indices_monitor"
      ]
    }
  ]
}
```

这个组合适用于：

- `GET /logs-2026.04/_settings`
- `GET /logs-2026.04/_stats`
- `GET /_cat/indices/logs-2026.04?format=json`

如果只需要修改 mapping，而不需要读写文档，可单独授予 `manage`：

```json
{
  "cluster": [],
  "indices": [
    {
      "names": [
        "logs-*"
      ],
      "query": "",
      "field_security": [],
      "field_mask": [],
      "privileges": [
        "manage"
      ]
    }
  ]
}
```

补充说明：

1. 如果只想把读取权限缩小到最小范围，可以使用更细粒度的 `search` 或 `get`，而不是直接使用 `read`。
2. 当前实现下，`search` 只覆盖搜索相关 API，不覆盖 `GET /_doc/<id>` 或 `POST /_mget`。
3. 当前实现下，`get` 只覆盖 `GET /_doc/<id>`，不覆盖 `_search` 或 `_mget`。
4. 当前实现下，`index` 或 `delete` 单独授予时，通常不足以支持常见 REST 写入；若要覆盖顶层 bulk 鉴权链路，请优先使用 `cluster_composite_ops + write` 或 `cluster_composite_ops + crud`。

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

响应示例：

```json
{
  "took": 3,
  "timed_out": false,
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.0,
    "hits": [
      {
        "_index": ".security",
        "_id": "a3f72d6f-2f9a-4a4d-89a0-4f261ee0d2ad",
        "_score": 0.0,
        "_source": {
          "token_doc_type": "access_token",
          "name": "backup-bot",
          "description": "token for snapshot inspection",
          "type": "general",
          "status": "active",
          "cluster": [
            "cluster_monitor"
          ],
          "indices": [
            {
              "names": [
                "snapshots*"
              ],
              "query": "",
              "field_security": [],
              "field_mask": [],
              "privileges": [
                "read"
              ]
            }
          ],
          "expire_in": 1772812800,
          "created": 1772809200000,
          "updated": 1772809200000
        }
      }
    ]
  }
}
```

使用请求体搜索（推荐用 `POST`）：

```bash
curl -k -u admin:admin -X POST "https://localhost:9200/_security/access_token/search" \
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

响应示例：

```json
{
  "took": 2,
  "timed_out": false,
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": ".security",
        "_id": "a3f72d6f-2f9a-4a4d-89a0-4f261ee0d2ad",
        "_score": 0.2876821,
        "_source": {
          "token_doc_type": "access_token",
          "name": "backup-bot-v2",
          "description": "updated token",
          "type": "general",
          "status": "active",
          "cluster": [
            "cluster_monitor"
          ],
          "indices": [
            {
              "names": [
                "snapshots*"
              ],
              "query": "",
              "field_security": [],
              "field_mask": [],
              "privileges": [
                "read"
              ]
            }
          ],
          "expire_in": 1772899200,
          "created": 1772809200000,
          "updated": 1772811000000
        }
      }
    ]
  }
}
```

说明：

1. 默认按 `created desc` 排序。
2. 默认 `size=10`。
3. `size` 最大为 `1000`。
4. 搜索接口返回的 `_source` 中不包含 `access_token` 字段；该字段会被服务端直接排除，不会以掩码、星号或加密字符串形式返回。后续更新或删除时，请使用结果中的文档 `_id` 作为 `token_id`。

## 更新 Token

更新接口路径中的 `{token_id}` 取自 token 字符串前 36 位 UUID，或者搜索结果中的文档 `_id`。

示例：

```bash
curl -k -u admin:admin -X PUT "https://localhost:9200/_security/access_token/a3f72d6f-2f9a-4a4d-89a0-4f261ee0d2ad" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "backup-bot-v2",
    "description": "updated token",
    "cluster": [
      "cluster_monitor"
    ],
    "indices": [
      {
        "names": [
          "snapshots*"
        ],
        "query": "",
        "field_security": [],
        "field_mask": [],
        "privileges": [
          "read"
        ]
      }
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
2. `description`、`expire_in` 可以按需更新；如果需要修改权限，请按完整的 `cluster` / `indices` 描述提交。
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
3. enrollment access token 的默认 TTL 为 10 年，但不是“签发后固定 10 年到期”。
4. 同一个 enroll token 重复成功调用 `/_cluster/enroll/node` 时，会复用同一个 access token，并重新计算 `expire_in` 写回，也就是按成功调用时间做滑动续期。
5. enrollment access token 的权限固定为 `cluster_monitor`，主要用于节点加入集群时的只读引导请求。

因此：

- 如果你的目标是让脚本或应用访问业务 API，请使用通用 Access Token。
- 如果你的目标是节点 enrollment，请使用 `/_cluster/enroll/node` 返回的 token。

## Enrollment token 中 `endpoint` / `endpoints` 的兼容说明

`POST /_cluster/enroll/token` 的顶层响应仍然是：

```json
{
  "token": "<base64>",
  "expires_at": "2026-06-10T10:39:11.167096570Z"
}
```

其中 `token` Base64 解码后的 payload 会包含 `endpoint` 与 `endpoints`：

```json
{
  "endpoint": "https://192.168.1.10:9200",
  "endpoints": [
    "https://192.168.1.10:9200",
    "https://es-node-2.example.com:9200",
    "https://[2001:db8::1]:9200"
  ],
  "cluster_name": "easysearch_test",
  "cluster_uuid": "3LvNNqCXQ1Kq2zn6wyuGwg",
  "access_token": "g5fHDWTknXHOR6wzhUnQ-APTGeMXgvUmdQ74yqu8DvI",
  "issued_at": "2026-06-10T10:09:11.167096570Z",
  "expires_at": "2026-06-10T10:39:11.167096570Z",
  "scope": "enroll:node"
}
```

说明：

1. `endpoint` 保留为**单个字符串**，只是为了兼容旧版本调用方的使用习惯。
2. `endpoints` 是当前推荐的新字段，包含节点发布的 HTTP publish address 列表。
3. `endpoint` 的值等于 `endpoints[0]`，仅作为兼容字段保留，不建议新调用方继续只依赖这个单值字段。
4. 新调用方应优先使用 `endpoints`，以便在首个地址不可用时尝试其他节点。
5. `endpoints` 中的成员来自节点的 HTTP publish address，因此成员可能是 IPv4、hostname 或 IPv6。
6. `endpoints` 只反映节点发布地址，调用方仍需确保这些地址对自身网络环境可达。
7. 当前实现统一按签发 token 的节点配置为地址添加协议：`security.ssl.http.enabled=true` 时使用 `https`，关闭时使用 `http`。
   如果集群各节点的 HTTP TLS 设置不同，应核对目标节点的实际协议；接口不会逐节点检查 TLS 开关。

## 获取节点 enrollment 配置

`POST /_cluster/enroll/node` 返回节点加入集群所需的配置、证书材料和只读引导用的 access token。这个请求用于获取引导信息，
实际启动新节点、写入配置和加入集群仍由调用方完成。

安全模块必须启用，且不能处于 `security.ssl_only=true` 的模式；仅 SSL 模式不会注册此接口。
HTTP TLS 可以关闭，节点间通信的 transport TLS 仍须开启并提供完整材料。

### 请求

先通过 `POST /_cluster/enroll/token` 获取 enrollment token，并按上节说明 Base64 解码顶层 `token`。
签发请求需要具有该安全 REST API 的访问权限；角色访问由 `security.restapi.roles_enabled` 及相应的接口禁用配置控制，也支持管理员证书认证。
取解码后 payload 的 `access_token` 作为请求头 `X-API-TOKEN`，请求体可以为空。
不要把顶层 Base64 字符串或通用 Access Token 直接作为此请求的 token。

```bash
curl -X POST "http://192.168.1.10:9200/_cluster/enroll/node" \
  -H "X-API-TOKEN: <解码后 payload 中的 access_token>"
```

上例对应 HTTP TLS 关闭的节点。开启时使用 payload 中返回的 `https` 地址，并为客户端配置相应 CA 信任。

### 成功响应

以下为 HTTP TLS 关闭时的主要响应字段示例，省略了其他安全配置、JDK 和健康信息。
尖括号中的值为占位符；证书字段实际返回 PEM 内容，包括 JSON 转义后的换行。

```json
{
  "cluster_name": "easysearch_test",
  "cluster_uuid": "3LvNNqCXQ1Kq2zn6wyuGwg",
  "security.enabled": true,
  "security.ssl.http.enabled": false,
  "security.ssl.transport.enabled": true,
  "security.ssl_only": false,
  "security.ssl.transport.cert_file": "<PEM 节点证书内容>",
  "security.ssl.transport.key_file": "<PEM 节点私钥内容>",
  "security.ssl.transport.ca_file": "<PEM CA 内容>",
  "seed_addresses": ["192.168.1.10:9300", "192.168.1.11:9300"],
  "version": "2.5.0",
  "plugins": [],
  "access_token": "<节点引导用的 access token>"
}
```

字段名中带点的键是**顶层字段名**，例如应读取 `response["security.ssl.transport.enabled"]`。
安全配置值来自处理请求的节点；未显式配置的开关按该配置的默认值返回。

| 字段 | 类型 | 说明 |
|------|------|------|
| `cluster_name`、`cluster_uuid` | 字符串 | 集群名称和唯一标识 |
| `security.enabled` | 布尔值 | 安全模块开关，配置默认值为 `true` |
| `security.ssl.http.enabled` | 布尔值 | HTTP TLS 开关，配置默认值为 `false` |
| `security.ssl.transport.enabled` | 布尔值 | Transport TLS 开关，配置默认值为 `true`；2.5.0 新增返回字段 |
| `security.ssl_only` | 布尔值 | 仅 SSL 模式开关，配置默认值为 `false`；2.5.0 新增返回字段。当前此接口在仅 SSL 模式下不可用 |
| `security.ssl.http_ca_file` | 字符串 | HTTP 信任材料的 PEM 内容，仅 HTTP TLS 开启时返回；优先取配置的 CA，存在回退到节点证书的情况，见下文 |
| `security.ssl.http.cert_file`、`security.ssl.http.key_file` | 字符串 | HTTP 节点证书、私钥的 PEM 内容，仅 HTTP TLS 开启时返回 |
| `security.ssl.transport.cert_file`、`security.ssl.transport.key_file`、`security.ssl.transport.ca_file` | 字符串 | Transport 节点证书、私钥、CA 的 PEM 内容，成功响应中必需 |
| `security.allow_default_init_securityindex` | 布尔值 | 默认安全索引初始化开关 |
| `security.nodes_dn` | 字符串数组 | 节点静态配置中的 DN 列表，不反映动态 nodes_dn API 管理的值 |
| `security.restapi.roles_enabled` | 字符串数组 | 允许访问安全 REST API 的角色 |
| `security.system_indices.enabled` | 布尔值 | 系统索引保护开关 |
| `security.system_indices.indices` | 字符串数组 | 系统索引模式 |
| `security.ssl.http.clientauth_mode` | 字符串 | HTTP 客户端证书认证模式；HTTP TLS 关闭时该字段仍返回，但不表示启用了客户端证书认证 |
| `security.authcz.admin_dn` | 字符串数组 | 管理员证书 DN 列表 |
| `seed_addresses` | 字符串数组 | 去重并排序后的节点 transport 发布地址，用于集群发现；不是 HTTP 访问地址 |
| `version`、`plugins` | 字符串、字符串数组 | 响应节点的版本及插件目录中的插件名称列表；内置 modules 不计入该列表 |
| `jdk` | 对象 | 响应节点的 JDK 信息，包含 `version` 和 `vendor` |
| `health` | 对象 | 当前集群状态，包含状态、节点数和分片统计 |
| `access_token` | 字符串 | 新签发或复用的 enrollment access token，权限与续期规则见上文；它与请求头中的 enrollment token 用途不同 |

### HTTP TLS 与证书材料处理

从 2.5.0 起，`security.ssl.http.enabled=false` 时不要求 HTTP CA、证书或私钥，响应会省略这三个 HTTP 材料字段，
而不是返回 `null` 或空字符串。只配置完整的 transport PEM 三件套也能获取 enrollment 配置。
调用方应先检查 HTTP TLS 开关，再按需读取 HTTP 材料；不要假定成功响应总包含它们。

HTTP TLS 开启时，这三个 HTTP 材料字段仍是必需的。任一材料经过回退后仍无法取得，或读取文件、加载 keystore/truststore 时发生异常，会返回 `500`。
HTTP 材料的读取顺序如下；使用 TLCP 时，相应的签名证书、签名私钥和可信 CA 文件优先于普通 PEM 配置。

| 响应材料 | 读取顺序 |
|----------|----------|
| HTTP 节点证书 | HTTP 证书文件 → HTTP keystore → transport keystore |
| HTTP 私钥 | HTTP 私钥文件 → HTTP keystore → transport keystore |
| HTTP CA 字段 | HTTP CA 文件 → HTTP 节点证书文件 → HTTP truststore → 按 HTTP 节点证书的读取规则回退 |

因此，未单独配置 HTTP CA 文件不一定报错；`security.ssl.http_ca_file` 可能包含回退取得的节点证书，不能仅根据字段名认定它是独立 CA 证书。
该响应字段名与节点配置项 `security.ssl.http.ca_file` 不同。接口不会自动把 transport PEM 三件套作为 HTTP 材料；
transport 材料在成功响应中始终必需。

回退针对没有取得材料的情况；已配置文件读取或 keystore 加载发生异常时，接口返回错误，不会忽略异常继续尝试后续来源。
文件形式的材料会被读取并转换为 PEM 文本，此接口不会完整验证证书链、有效期或私钥与证书是否匹配；
不能以 enrollment 请求成功代替新节点的 TLS 配置验证。

这些 `*_file` 字段返回的是**证书或私钥内容，不是源节点上的文件路径**。
如果调用方据此生成新节点配置，需要把内容写到相应文件，并在新节点的配置项中填写文件路径。

### 错误响应

以下为节点 enrollment 处理逻辑返回的错误；HTTP 层的认证或请求检查也可能在进入该逻辑前返回错误。

| HTTP | `error.type` | 场景 |
|------|--------------|------|
| `400` | `cluster_mismatch_exception` | token 对应的集群名称或 UUID 与当前集群不一致 |
| `401` | `invalid_token_exception` | 缺少 `X-API-TOKEN`，或 token 无效、记录已被清理 |
| `403` | `forbidden_token_scope` | token 的 scope 不允许节点 enrollment |
| `410` | `token_expired_exception` | 找到了 token 记录，但已过期；检测到过期后尝试删除记录 |
| `503` | `service_unavailable` | 集群 UUID 尚未提交 |
| `500` | `internal_server_error` | 必需材料缺失、文件无法读取或解析，或签发引导 access token 时发生内部错误 |

过期 token 如果已被定时清理或先前的校验删除，再次请求会返回 `401`；调用方需要重新签发 enrollment token。

例如，HTTP TLS 开启且 HTTP CA 字段按上述回退规则仍无法取得时：

```json
{
  "error": {
    "type": "internal_server_error",
    "reason": "missing enrollment material: security.ssl.http_ca_file"
  },
  "status": 500
}
```

## 实践建议

1. 为不同调用方签发不同 token，不要多人或多服务共用一个 token。
2. `expire_in` 尽量设置较短，并定期轮换。
3. 优先授予最小权限集合。
4. 对高权限 token 保留审计与使用记录。
5. 如果只需要简单的管理操作，优先考虑通过角色体系管理用户，而不是长期使用高权限 token。

