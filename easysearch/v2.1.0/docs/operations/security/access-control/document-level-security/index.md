---
title: "文档级权限"
date: 0001-01-01
summary: "文档级权限 #  文档级权限允许您将角色限制为索引中文档的一部分子集。
相关指南（先读这些） #    权限控制总览  安全与多租户最佳实践  参考设置 #  文档级权限使用 Easysearch 查询 DSL 来定义角色授予对哪些文档的访问权限。
{ &#34;bool&#34;: { &#34;must&#34;: { &#34;match&#34;: { &#34;public&#34;: &#34;true&#34; } } } } 上面的查询指定了该角色访问的文档里面，其字段 public 必须匹配 true。
指定字段 query 并设置为将上面的查询，并对查询字符串进行转义，最后的角色设置如下：
PUT _security/role/public_data { &#34;cluster&#34;: [ &#34;*&#34; ], &#34;indices&#34;: [{ &#34;names&#34;: [ &#34;pub*&#34; ], &#34;query&#34;: &#34;{\&#34;term\&#34;: { \&#34;public\&#34;: true}}&#34;, &#34;privileges&#34;: [ &#34;read&#34; ] }] } 上面的查询也可以根据需要写的很复杂，但是我们建议保持简单，以最大程度地减少文档级安全功能对集群的性能影响。
参数替换 #  查询过程中可以利用上下文变量，可根据当前用户的属性来强制实施规则替换。例如 ${user."
---


# 文档级权限

文档级权限允许您将角色限制为索引中文档的一部分子集。

## 相关指南（先读这些）

- [权限控制总览]({{< relref "/docs/operations/security/access-control/_index.md" >}})
- [安全与多租户最佳实践]({{< relref "/docs/best-practices/security-and-multi-tenancy.md" >}})

## 参考设置

文档级权限使用 Easysearch 查询 DSL 来定义角色授予对哪些文档的访问权限。

```json
{
  "bool": {
    "must": {
      "match": {
        "public": "true"
      }
    }
  }
}
```

上面的查询指定了该角色访问的文档里面，其字段 `public` 必须匹配 `true`。

指定字段 `query` 并设置为将上面的查询，并对查询字符串进行转义，最后的角色设置如下：

```json
PUT _security/role/public_data
{
  "cluster": [
    "*"
  ],
  "indices": [{
    "names": [
      "pub*"
    ],
    "query": "{\"term\": { \"public\": true}}",
    "privileges": [
      "read"
    ]
  }]
}
```

上面的查询也可以根据需要写的很复杂，但是我们建议保持简单，以最大程度地减少文档级安全功能对集群的性能影响。

## 参数替换

查询过程中可以利用上下文变量，可根据当前用户的属性来强制实施规则替换。例如 `${user.name}` 将替换为当前用户的名称。

如下规则允许用户读取字段 `readable_by` 为其用户名值的任何文档：

```json
PUT _security/role/user_data
{
  "cluster": [
    "*"
  ],
  "indices": [{
    "names": [
      "pub*"
    ],
    "query": "{\"term\": { \"readable_by\": \"${user.name}\"}}",
    "privileges": [
      "read"
    ]
  }]
}
```

支持以下变量：

| 名称                    | 描述                                                    |
| :---------------------- | :------------------------------------------------------ |
| `${user.name}`          | 用户名.                                                 |
| `${user.roles}`         | 用户的角色列表，使用逗号分割。                          |
| `${attr.<TYPE>.<NAME>}` | 用户的自定义属性， `<TYPE>` 支持 `internal` 和 `ldap`。 |

## 基于属性的权限控制

可以将角色和参数替换与 `terms_set` 查询一起使用，以启用基于属性的权限控制。

#### 定义角色

```json
PUT _security/role/abac
{
  "indices": [{
    "names": [
      "*"
    ],
    "query": "{\"terms_set\": {\"security_attributes\": {\"terms\": [${attr.internal.permissions}], \"minimum_should_match_script\": {\"source\": \"doc['security_attributes'].length\"}}}}",
    "privileges": [
      "read"
    ]
  }]
}
```

#### 定义用户

```json
PUT _security/user/user1
{
  "password": "asdf",
  "roles": ["abac"],
  "attributes": {
    "permissions": "\"att1\", \"att2\", \"att3\""
  }
}
```

#### 定义文档

```
POST easysearch/_doc/1
{
  "security_attributes":"att1"
}
```

> 注意，索引的 `security_attributes` 必须是 `keyword` 类型。

