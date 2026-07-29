---
title: "系统索引"
date: 0001-01-01
summary: "系统索引 #  Easysearch 默认的身份信息存放在一个受保护的系统索引里面，名称为：.security， 将索引设置为系统索引可以对该索引的数据进行额外的保护，因为即使您的用户帐户对所有索引具有读取权限，也无法直接访问此系统索引中的数据。
您可以在 easysearch.yml 中添加其它您希望需要受到保护的索引。
security.system_indices.enabled: true security.system_indices.indices: [&#34;.infini-*&#34;] 如果要访问系统索引，必须使用管理员证书的方式来进行： 配置管理证书:
curl -k --cert ./admin.crt --key ./admin.key -XGET &#39;https://localhost:9200/.security/_search&#39; 另一种方法是从每个节点上的 security.system_indices.index 列表中删除该索引，然后重新启动 Easysearch 即可正常操作该索引。"
---


# 系统索引

Easysearch 默认的身份信息存放在一个受保护的系统索引里面，名称为：`.security`，
将索引设置为系统索引可以对该索引的数据进行额外的保护，因为即使您的用户帐户对所有索引具有读取权限，也无法直接访问此系统索引中的数据。

您可以在 `easysearch.yml` 中添加其它您希望需要受到保护的索引。

```yml
security.system_indices.enabled: true
security.system_indices.indices: [".infini-*"]
```

如果要访问系统索引，必须使用管理员证书的方式来进行：[配置管理证书](./tls.md#配置管理证书):

```bash
curl -k --cert ./admin.crt --key ./admin.key -XGET 'https://localhost:9200/.security/_search'
```

另一种方法是从每个节点上的 `security.system_indices.index` 列表中删除该索引，然后重新启动 Easysearch 即可正常操作该索引。

