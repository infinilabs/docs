---
title: "字段级权限"
date: 0001-01-01
summary: "字段级权限 #  字段级权限允许您控制用户可以查看文档中的哪些字段。就像 文档级权限，可以通过角色配置中的索引块来控制访问。
相关指南（先读这些） #    权限控制总览  安全与多租户最佳实践  包括或排除字段 #  配置字段级权限时，有两个选项：包括或排除字段。如果包含字段，则用户在检索文档时 只能看到 这些字段。例如，如果您包含 actors、title 和 year 字段，则搜索结果可能如下所示：
POST movies/_doc/1 { &#34;year&#34;: 2013, &#34;title&#34;: &#34;Rush&#34;, &#34;actors&#34;: [ &#34;Daniel Brühl&#34;, &#34;Chris Hemsworth&#34;, &#34;Olivia Wilde&#34; ] } 如果是排除字段，则用户在检索文档时会看到除这些字段之外的所有内容。例如，如果排除这些相同的字段，则相同的搜索结果可能如下所示：
POST movies/_doc/2 { &#34;directors&#34;: [ &#34;Ron Howard&#34; ], &#34;plot&#34;: &#34;A re-creation of the merciless 1970s rivalry between Formula One rivals James Hunt and Niki Lauda.&#34;, &#34;genres&#34;: [ &#34;Action&#34;, &#34;Biography&#34;, &#34;Drama&#34;, &#34;Sport&#34; ] } 您可以使用配置文件 role."
---


# 字段级权限

字段级权限允许您控制用户可以查看文档中的哪些字段。就像[文档级权限](./document-level-security.md)，可以通过角色配置中的索引块来控制访问。

## 相关指南（先读这些）

- [权限控制总览]({{< relref "/docs/operations/security/access-control/_index.md" >}})
- [安全与多租户最佳实践]({{< relref "/docs/best-practices/security-and-multi-tenancy.md" >}})

## 包括或排除字段

配置字段级权限时，有两个选项：包括或排除字段。如果包含字段，则用户在检索文档时 _只能看到_ 这些字段。例如，如果您包含 `actors`、`title` 和 `year` 字段，则搜索结果可能如下所示：

```json
POST movies/_doc/1
{
    "year": 2013,
    "title": "Rush",
    "actors": [
      "Daniel Brühl",
      "Chris Hemsworth",
      "Olivia Wilde"
    ]
}
```

如果是排除字段，则用户在检索文档时会看到除这些字段之外的所有内容。例如，如果排除这些相同的字段，则相同的搜索结果可能如下所示：

```json
POST movies/_doc/2
{
  "directors": [
    "Ron Howard"
  ],
  "plot": "A re-creation of the merciless 1970s rivalry between Formula One rivals James Hunt and Niki Lauda.",
  "genres": [
    "Action",
    "Biography",
    "Drama",
    "Sport"
  ]
}
```

您可以使用配置文件 `role.yml` 和 REST API 来指定字段级安全设置。

- 要排除字段，可以在配置 `role.yml` 或通过 REST API，在字段名称前添加 `~` 。
- 字段名称支持通配符 (`*`).

### role.yml

```yml
limited_movie:
  cluster: []
  indices:
    - names:
        - movies
      privileges:
        - "read"
      field_security:
        - "~actors"
        - "~title"
        - "~year"
```

### REST API

参照[创建角色](./api.md#创建一个角色).

## 注意多个角色

如果将用户映射到多个角色，我们建议这些角色对每个索引使用 include _or_ exclude 语句。安全模块使用 `AND` 运算符来评估字段级权限的设置，因此组合包含和排除语句可能会导致这两种行为都无法正常工作。

例如，在 `movies` 索引中，如果您在一个角色中包含 `actors` 、 `title` 和 `year`，在另一个角色中排除 `actors` 、 `title` 和 `genres`，然后将这两个角色映射到同一用户，则搜索结果可能如下所示：

```json
{
  "_index": "movies",
  "_type": "_doc",
  "_source": {
    "year": 2013,
    "directors": ["Ron Howard"],
    "plot": "A re-creation of the merciless 1970s rivalry between Formula One rivals James Hunt and Niki Lauda."
  }
}
```

## 与文档级权限的交互

[文档级权限](./document-level-security.md) 依赖于 Easysearch 查询，这意味着查询中的所有相关字段都必须可见才能正常工作。如果将字段级权限与文档级权限结合使用，请确保不要限制对文档级权限字段的访问。

