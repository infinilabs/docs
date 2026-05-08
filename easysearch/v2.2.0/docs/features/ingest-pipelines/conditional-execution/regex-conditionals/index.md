---
title: "正则表达式条件"
date: 0001-01-01
summary: "正则表达式条件 #  摄取管道支持使用正则表达式（regex）和 Painless 脚本语言进行条件逻辑。这允许根据文本字段的结构和内容对哪些文档进行处理进行精细控制。正则表达式可以用于 if 参数中评估字符串模式。这对于匹配 IP 格式、验证电子邮件地址、识别 UUID 或处理包含特定关键字的日志特别有用。
示例：电子邮件名过滤 #  以下管道使用正则表达式来识别来自 @example.com 电子邮件域的用户，并相应地标记这些文档：
PUT _ingest/pipeline/tag_example_com_users { &#34;processors&#34;: [ { &#34;set&#34;: { &#34;field&#34;: &#34;user_domain&#34;, &#34;value&#34;: &#34;example.com&#34;, &#34;if&#34;: &#34;ctx.email != null &amp;&amp; ctx.email =~ /@example.com$/&#34; } } ] } 使用以下请求来模拟管道：
POST _ingest/pipeline/tag_example_com_users/_simulate { &#34;docs&#34;: [ { &#34;_source&#34;: { &#34;email&#34;: &#34;alice@example.com&#34; } }, { &#34;_source&#34;: { &#34;email&#34;: &#34;bob@another.com&#34; } } ] } 只有第一份文档添加了 user_domain ：
{ &#34;docs&#34;: [ { &#34;doc&#34;: { &#34;_source&#34;: { &#34;email&#34;: &#34;alice@example."
---


# 正则表达式条件

摄取管道支持使用正则表达式（`regex`）和 Painless 脚本语言进行条件逻辑。这允许根据文本字段的结构和内容对哪些文档进行处理进行精细控制。正则表达式可以用于 `if` 参数中评估字符串模式。这对于匹配 IP 格式、验证电子邮件地址、识别 UUID 或处理包含特定关键字的日志特别有用。

## 示例：电子邮件名过滤

以下管道使用正则表达式来识别来自 `@example.com` 电子邮件域的用户，并相应地标记这些文档：

```
PUT _ingest/pipeline/tag_example_com_users
{
  "processors": [
    {
      "set": {
        "field": "user_domain",
        "value": "example.com",
        "if": "ctx.email != null && ctx.email =~ /@example.com$/"
      }
    }
  ]
}
```

使用以下请求来模拟管道：

```
POST _ingest/pipeline/tag_example_com_users/_simulate
{
  "docs": [
    { "_source": { "email": "alice@example.com" } },
    { "_source": { "email": "bob@another.com" } }
  ]
}
```

只有第一份文档添加了 `user_domain` ：

```
{
  "docs": [
    {
      "doc": {
        "_source": {
          "email": "alice@example.com",
          "user_domain": "example.com"
        }
      }
    },
    {
      "doc": {
        "_source": {
          "email": "bob@another.com"
        }
      }
    }
  ]
}
```

## 示例：检测 IPv6 地址

以下管道使用正则表达式来识别并标记 IPv6 格式的地址：

```
PUT _ingest/pipeline/ipv6_flagger
{
  "processors": [
    {
      "set": {
        "field": "ip_type",
        "value": "IPv6",
        "if": "ctx.ip != null && ctx.ip =~ /^[a-fA-F0-9:]+$/ && ctx.ip.contains(':')"
      }
    }
  ]
}
```

使用以下请求来模拟管道：

```
POST _ingest/pipeline/ipv6_flagger/_simulate
{
  "docs": [
    { "_source": { "ip": "2001:0db8:85a3:0000:0000:8a2e:0370:7334" } },
    { "_source": { "ip": "192.168.0.1" } }
  ]
}
```

第一个文档包含一个添加的 `ip_type` 字段，设置为 `IPv6` ：

```

{
  "docs": [
    {
      "doc": {
        "_source": {
          "ip": "2001:0db8:85a3:0000:0000:8a2e:0370:7334",
          "ip_type": "IPv6"
        }
      }
    },
    {
      "doc": {
        "_source": {
          "ip": "192.168.0.1"
        }
      }
    }
  ]
}
```

## 示例：验证 UUID 字符串

以下管道使用正则表达式来验证 `session_id` 字段是否包含有效的 UUID：

```
PUT _ingest/pipeline/uuid_checker
{
  "processors": [
    {
      "set": {
        "field": "valid_uuid",
        "value": true,
        "if": "ctx.session_id != null && ctx.session_id =~ /^[a-f0-9]{8}-[a-f0-9]{4}-4[a-f0-9]{3}-[89ab][a-f0-9]{3}-[a-f0-9]{12}$/"
      }
    }
  ]
}
```

使用以下请求来模拟管道：

```
POST _ingest/pipeline/uuid_checker/_simulate
{
  "docs": [
    { "_source": { "session_id": "550e8400-e29b-41d4-a716-446655440000" } },
    { "_source": { "session_id": "invalid-uuid-1234" } }
  ]
}
```

第一份文档被标记为新的 `valid_uuid` 字段：

```
{
  "docs": [
    {
      "doc": {
        "_source": {
          "session_id": "550e8400-e29b-41d4-a716-446655440000",
          "valid_uuid": true
        }
      }
    },
    {
      "doc": {
        "_source": {
          "session_id": "invalid-uuid-1234"
        }
      }
    }
  ]
}
```

