---
title: "复制 curl 命令"
date: 0001-01-01
description: "在 Easysearch UI 开发工具中复制 curl 命令。"
summary: "复制 curl 命令 #  操作步骤 #    进入开发工具页面：在左侧导航栏点击「开发工具」，进入开发工具编辑页。
  编写请求语句：在左侧编辑区输入符合规范的请求语句。
  生成并复制 curl 命令：点击编辑区右上角的「curl 命令」按钮，系统将自动生成对应 curl 命令并复制到剪贴板。
   查看复制结果：  curl -XGET &#34;http://xxx.xxx.xxx/_search&#34; -H &#39;Content-Type: application/json&#39; -d&#39; { &#34;query&#34;: { &#34;match_all&#34;: {} } }&#39; 补充说明 #   复制的 curl 命令中会自动包含请求方法、地址、请求头和请求体，可直接在终端中执行。 如果集群启用了 HTTPS 或认证，请在粘贴后根据实际情况补充 -k（跳过证书校验）或 -u user:password（认证信息）等参数。  "
---


# 复制 curl 命令

## 操作步骤

1. **进入开发工具页面**：在左侧导航栏点击「开发工具」，进入开发工具编辑页。

2. **编写请求语句**：在左侧编辑区输入符合规范的请求语句。

3. **生成并复制 curl 命令**：点击编辑区右上角的「curl 命令」按钮，系统将自动生成对应 curl 命令并复制到剪贴板。

![](/img/easysearch-ui/dev-tools/copy-curl-command/image-1.png)

4. **查看复制结果**：

```bash
curl -XGET "http://xxx.xxx.xxx/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}'
```

## 补充说明

- 复制的 curl 命令中会自动包含请求方法、地址、请求头和请求体，可直接在终端中执行。
- 如果集群启用了 HTTPS 或认证，请在粘贴后根据实际情况补充 `-k`（跳过证书校验）或 `-u user:password`（认证信息）等参数。

