---
title: "DBX 轻量桌面客户端"
date: 0001-01-01
description: "DBX 是一款开源、轻量的数据库工具，支持 70 多种数据库与数据服务。从 DBX v0.5.71 正式支持支持 Easysearch 连接管理、索引浏览、文档操作与查询。"
summary: "DBX 轻量桌面客户端 #  DBX 是一款开源、轻量的数据库工具，支持 70 多种数据库与数据服务。针对 Easysearch，DBX 提供以下常用能力：
 保存并管理 Easysearch 连接，支持用户名、密码、TLS/SSL 与代理配置； 展开连接后直接浏览当前账号有权访问的索引； 以表格或文档视图查看、筛选、排序和分页浏览文档； 在查询编辑器中执行 REST、Query DSL 与常用 SQL； 对具备写入权限的索引新增、修改或删除文档； 通过 DBX MCP 将已有连接提供给支持 MCP 的 AI 工具使用。  在 Windows、macOS、Linux 或 Docker 环境中，使用一个轻量桌面客户端完成 Easysearch 连接管理、索引浏览、文档操作与查询。
三步连接 Easysearch #   本文截图与示例基于 DBX v0.5.71 和 Easysearch v2.3.1。连接测试、集群状态查询、索引浏览、文档搜索与写入能力均已在实际环境中完成验证。
 1. 选择 Easysearch #  打开 DBX，点击顶部的“新建连接”。在数据库类型列表中搜索并选择 Easysearch，然后点击“下一步”。
图 1 在 DBX 中选择 Easysearch 2. 填写连接信息 #  填写连接名称、主机、端口、用户名和密码。Easysearch 常用 HTTP 端口为 9200；如果服务使用 HTTPS，请在“TLS/SSL”页签中配置证书校验方式。"
---


# DBX 轻量桌面客户端

DBX 是一款开源、轻量的数据库工具，支持 70 多种数据库与数据服务。针对 Easysearch，DBX 提供以下常用能力：

- 保存并管理 Easysearch 连接，支持用户名、密码、TLS/SSL 与代理配置；
- 展开连接后直接浏览当前账号有权访问的索引；
- 以表格或文档视图查看、筛选、排序和分页浏览文档；
- 在查询编辑器中执行 REST、Query DSL 与常用 SQL；
- 对具备写入权限的索引新增、修改或删除文档；
- 通过 DBX MCP 将已有连接提供给支持 MCP 的 AI 工具使用。

在 Windows、macOS、Linux 或 Docker 环境中，使用一个轻量桌面客户端完成 Easysearch 连接管理、索引浏览、文档操作与查询。

## 三步连接 Easysearch

> 本文截图与示例基于 **DBX v0.5.71** 和 **Easysearch v2.3.1**。连接测试、集群状态查询、索引浏览、文档搜索与写入能力均已在实际环境中完成验证。

### 1. 选择 Easysearch

打开 DBX，点击顶部的“新建连接”。在数据库类型列表中搜索并选择 Easysearch，然后点击“下一步”。

{{% load-img "/img/integrations/clients/dbx/1.png" %}}   

<center><small>图 1  在 DBX 中选择 Easysearch</small></center>

### 2. 填写连接信息

填写连接名称、主机、端口、用户名和密码。Easysearch 常用 HTTP 端口为 9200；如果服务使用 HTTPS，请在“TLS/SSL”页签中配置证书校验方式。

也可以直接粘贴完整连接 URL，由 DBX 自动解析连接参数。发布或分享截图时，请始终隐藏真实密码、内部地址和访问令牌。

{{% load-img "/img/integrations/clients/dbx/2.png" %}}

<center><small>图 2  填写 Easysearch 连接信息</small></center>

### 3. 测试并保存

点击“测试”确认网络、账号和权限配置正确，再点击“保存并连接”。连接成功后，DBX 会在左侧连接树中展示当前账号可访问的索引。

> **连接失败时优先检查**    
> 服务地址与端口是否可达；用户名、密码和账号权限是否正确；HTTPS 证书链与主机名校验是否匹配；本机代理、隧道或防火墙是否允许访问目标服务。

## 浏览索引与文档

展开 Easysearch 连接并点击索引，DBX 会读取索引结构并加载文档。常见字段可以直接以表格形式展示，也可以切换到文档视图查看完整 JSON。

{{% load-img "/img/integrations/clients/dbx/3.png" %}}
<center><small>图 3  在 DBX 中浏览 Easysearch 索引文档</small></center>

工具栏提供刷新、自动刷新、跳转列、新增行、提交和回滚等操作。对于具备写入权限的索引，可以直接在结果区域新增或修改文档，再统一提交变更。

需要缩小结果范围时，可以在筛选区域输入 JSON 条件。例如筛选指定分类：

```json
{
  "category": "compatibility"
}
```

需要使用完整 Query DSL 时，可以通过 $esQuery 传入查询对象：

```json
{
  "$esQuery": {
    "match": {
      "title": "Easysearch"
    }
  }
}
```

## 执行 REST、Query DSL 与 SQL

DBX 的 Easysearch 查询编辑器支持 METHOD /path 格式的 REST 请求。输入请求后，使用编辑器左侧的执行按钮或快捷键运行，即可在下方查看状态码和 JSON 响应。

例如，检查集群健康状态：

```
GET /_cluster/health
```

{{% load-img "/img/integrations/clients/dbx/4.png" %}}
<center><small>图 4  在 DBX 中执行 Easysearch REST 查询</small></center>


## 更多查询方式

执行 Query DSL：

```
POST /products/_search
{
  "query": {
    "match": {
      "name": "database"
    }
  }
}
```


对于 Easysearch 当前版本支持的常用 SQL，也可以直接输入：

```sql
SELECT title, category, score   FROM products   ORDER BY score DESC   LIMIT 20; 
```

DBX 会将兼容响应转换为结果表格。遇到 Elasticsearch 版本差异、专有扩展或 SQL 兼容性限制时，建议改用明确的 REST 与 Query DSL 请求，以便准确控制请求路径和参数。

## 与 AI 工具协作

启用 DBX MCP 后，支持 MCP 的 AI 编码工具可以复用 DBX 中保存的 Easysearch 连接，在现有访问策略下执行查询，无需在提示词或聊天记录中重复填写连接密码。

> **生产环境建议**    
> 为 AI 工具单独准备只读或受限账号，明确允许访问的索引范围，并保持文档写入、删除索引等高风险操作默认关闭。

## 下载与反馈

DBX 官网：[https://dbxio.com](https://dbxio.com)  
DBX 下载：[https://github.com/t8y2/dbx/releases](https://github.com/t8y2/dbx/releases)  
DBX GitHub：[https://github.com/t8y2/dbx](https://github.com/t8y2/dbx)  

如果你正在使用 Easysearch，可以下载 DBX 体验连接管理、索引浏览、文档操作和查询能力。遇到兼容性问题或有新的使用建议，欢迎在 DBX GitHub 仓库提交 Issue。
