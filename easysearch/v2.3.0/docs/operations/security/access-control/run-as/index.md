---
title: "身份模拟"
date: 0001-01-01
summary: "身份模拟 #  用户模拟允许具备特定权限的用户以另外的身份来进行集群的访问。
用户模拟可用于测试和故障排除，或允许系统服务安全地充当用户。
在 REST 接口或 TCP 传输层上都可以进行用户模拟。
REST 接口 #  要允许一个用户模拟另一个用户，请将以下内容添加到 easysearch.yml :
security.authcz.rest_impersonation_user: &lt;AUTHENTICATED_USER&gt;: - &lt;IMPERSONATED_USER_1&gt; - &lt;IMPERSONATED_USER_2&gt; 模拟用户字段支持通配符。将其设置为 * 允许 AUTHENTICATED_USER 来模拟任意用户。
传输层配置 #  类似的配置方法如下：
security.authcz.impersonation_dn: &#34;CN=spock,OU=client,O=client,L=Test,C=DE&#34;: - worf 模拟其他用户 #  要模拟其他用户，请向系统提交请求，并将 HTTP 标头 security_run_as 设置为要模拟的用户的名称。例如：
curl -XGET -u &#39;admin:xxxxxxxxxxxx&#39; -k -H &#34;security_run_as: user_1&#34; https://localhost:9200/_security/authinfo?pretty "
---


# 身份模拟

用户模拟允许具备特定权限的用户以另外的身份来进行集群的访问。

用户模拟可用于测试和故障排除，或允许系统服务安全地充当用户。

在 REST 接口或 TCP 传输层上都可以进行用户模拟。

## REST 接口

要允许一个用户模拟另一个用户，请将以下内容添加到 `easysearch.yml` :

```yml
security.authcz.rest_impersonation_user:
  <AUTHENTICATED_USER>:
    - <IMPERSONATED_USER_1>
    - <IMPERSONATED_USER_2>
```

模拟用户字段支持通配符。将其设置为 `*` 允许 `AUTHENTICATED_USER` 来模拟任意用户。

## 传输层配置

类似的配置方法如下：

```yml
security.authcz.impersonation_dn:
  "CN=spock,OU=client,O=client,L=Test,C=DE":
    - worf
```

## 模拟其他用户

要模拟其他用户，请向系统提交请求，并将 HTTP 标头 `security_run_as` 设置为要模拟的用户的名称。例如：

```bash
curl -XGET -u 'admin:xxxxxxxxxxxx' -k -H "security_run_as: user_1" https://localhost:9200/_security/authinfo?pretty
```

