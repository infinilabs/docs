---
title: "数据脱敏"
date: 0001-01-01
summary: "数据脱敏 #  如果您的数据里面包含一些敏感信息，除了通过 字段级权限 来进行访问控制，您还可以通过混淆字段里面的内容来进行脱敏。目前，字段数据脱敏仅适用于基于字符串的字段，支持加密哈希和正则替换字段的内容。
字段脱敏与字段级权限一起可以在相同的角色级别和索引级别上工作。您可以允许某些角色查看明文格式的敏感字段，并为其他角色脱敏这些字段。带有脱敏字段的搜索结果可能如下所示：
{ &#34;_index&#34;: &#34;movies&#34;, &#34;_type&#34;: &#34;_doc&#34;, &#34;_source&#34;: { &#34;year&#34;: 2013, &#34;directors&#34;: [&#34;Ron Howard&#34;], &#34;title&#34;: &#34;ca998e768dd2e6cdd84c77015feb29975f9f498a472743f159bec6f1f1db109e&#34; } } 设置盐值 #  可以在 easysearch.yml 设置一个随机字符串:
security.compliance.salt: abcdefghijklmnopqrstuvqxyz1234567890    属性 说明     security.compliance.salt 生成哈希值时要使用的盐值。必须至少为 32 个字符。仅允许使用 ASCII 字符。选填。    配置脱敏字段 #  role.yml #  masked_movie: cluster: [] indices: - names: - movies privileges: - &#34;read&#34; field_mask: - &#34;genres&#34; - &#34;title&#34; REST API #  参照 创建角色."
---


# 数据脱敏

如果您的数据里面包含一些敏感信息，除了通过[字段级权限](./field-level-security.md) 来进行访问控制，您还可以通过混淆字段里面的内容来进行脱敏。目前，字段数据脱敏仅适用于基于字符串的字段，支持加密哈希和正则替换字段的内容。

字段脱敏与字段级权限一起可以在相同的角色级别和索引级别上工作。您可以允许某些角色查看明文格式的敏感字段，并为其他角色脱敏这些字段。带有脱敏字段的搜索结果可能如下所示：

```json
{
  "_index": "movies",
  "_type": "_doc",
  "_source": {
    "year": 2013,
    "directors": ["Ron Howard"],
    "title": "ca998e768dd2e6cdd84c77015feb29975f9f498a472743f159bec6f1f1db109e"
  }
}
```

## 设置盐值

可以在 `easysearch.yml` 设置一个随机字符串:

```yml
security.compliance.salt: abcdefghijklmnopqrstuvqxyz1234567890
```

| 属性                       | 说明                                                                          |
| :------------------------- | :---------------------------------------------------------------------------- |
| `security.compliance.salt` | 生成哈希值时要使用的盐值。必须至少为 32 个字符。仅允许使用 ASCII 字符。选填。 |

## 配置脱敏字段

### role.yml

```yml
masked_movie:
  cluster: []
  indices:
    - names:
        - movies
      privileges:
        - "read"
      field_mask:
        - "genres"
        - "title"
```

### REST API

参照[创建角色](./api.md#创建一个角色).

## 使用其它哈希算法

默认情况下，安全模块使用 BLAKE2b 算法，但您可以使用 JVM 提供的任何哈希算法。此列表通常包括 MD5、SHA-1、SHA-384 和 SHA-512。

要指定其它算法，请将其添加到脱敏字段之后：

```yml
masked_movie:
  cluster: []
  indices:
    - names:
        - movies
      privileges:
        - "read"
      field_mask:
        - "genres::SHA-512"
        - "title"
```

## 基于规则的字段脱敏

除了使用哈希，您还可以使用一个或多个正则表达式来替换字符串从而达到字段脱敏的效果。语法是 `<field>::/<regular-expression>/::<replacement-string>` 。如果使用多个正则表达式，则结果将从左向右传递，就像 shell 中的管道操作一样：

```yml
masked_movie:
  cluster: []
  indices:
    - names:
        - movies
      privileges:
        - "read"
      field_mask:
        - "genres::/^[a-zA-Z]{1,3}/::XXX::/[a-zA-Z]{1,3}$/::YYY"
        - "title::/./::*"
```

`title` 字段里面的表达式将字段中的每个字符更改为 `*`，因此您仍然可以辨别脱敏后字符串的长度。`genres` 字段的表达式将字符串的前三个字符更改为 `XXX`，将最后三个字符更改为 `YYY`。

完整测试脚本如下：

```bash
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

PUT _security/user/medcl
{
  "password": "pass",
  "roles": ["masked_movie"]
}

curl -XGET -k  'https://localhost:9200/movies/_search?pretty' -u medcl:pass

{
  "took" : 27,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "actors" : [
            "Daniel Brühl",
            "Chris Hemsworth",
            "Olivia Wilde"
          ],
          "year" : 2013,
          "title" : "****"
        }
      },
      {
        "_index" : "movies",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "plot" : "A re-creation of the merciless 1970s rivalry between Formula One rivals James Hunt and Niki Lauda.",
          "genres" : [
            "XXXYYY",
            "XXXgraYYY",
            "XXYYY",
            "XXYYY"
          ],
          "directors" : [
            "Ron Howard"
          ]
        }
      }
    ]
  }
}
```

## 对审计日志的影响

读取历史记录可让您跟踪对文档中敏感字段的读取操作。例如，您可以跟踪对客户记录的电子邮件字段的访问。对脱敏字段的访问从读取历史记录中排除了，因为用户只能看到哈希值，而不是字段的明文值。

