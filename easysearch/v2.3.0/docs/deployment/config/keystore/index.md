---
title: "Keystore 安全设置"
date: 0001-01-01
summary: "Keystore 安全设置 #  Easysearch 的 keystore 是一个加密的安全存储，用于保存敏感配置项（如密码、密钥、凭据等）。使用 keystore 可以避免将敏感信息以明文形式写在 easysearch.yml 配置文件中。
为什么使用 Keystore #     方式 安全性 说明     easysearch.yml ⚠️ 明文 配置文件中的密码任何有文件读取权限的用户都能看到   环境变量 ⚠️ 明文 环境变量可能在进程列表、日志中暴露   Keystore ✅ 加密 AES-GCM 加密存储，可选密码保护    适合存入 keystore 的设置示例：
 S3 快照仓库的 access_key 和 secret_key LDAP 绑定密码 安全插件的管理员密码 任何包含密码或密钥的配置项  Keystore 文件 #  keystore 文件名为 easysearch.keystore，存放在配置目录（$ES_PATH_CONF）下。文件权限自动设置为 rw-rw----（660）。
创建 Keystore #  bin/easysearch-keystore create 创建带密码保护的 keystore："
---


# Keystore 安全设置

Easysearch 的 keystore 是一个加密的安全存储，用于保存敏感配置项（如密码、密钥、凭据等）。使用 keystore 可以避免将敏感信息以明文形式写在 `easysearch.yml` 配置文件中。

## 为什么使用 Keystore

| 方式 | 安全性 | 说明 |
|------|--------|------|
| `easysearch.yml` | ⚠️ 明文 | 配置文件中的密码任何有文件读取权限的用户都能看到 |
| 环境变量 | ⚠️ 明文 | 环境变量可能在进程列表、日志中暴露 |
| **Keystore** | ✅ 加密 | AES-GCM 加密存储，可选密码保护 |

适合存入 keystore 的设置示例：
- S3 快照仓库的 `access_key` 和 `secret_key`
- LDAP 绑定密码
- 安全插件的管理员密码
- 任何包含密码或密钥的配置项

## Keystore 文件

keystore 文件名为 `easysearch.keystore`，存放在配置目录（`$ES_PATH_CONF`）下。文件权限自动设置为 `rw-rw----`（660）。

## 创建 Keystore

```bash
bin/easysearch-keystore create
```

创建带密码保护的 keystore：

```bash
bin/easysearch-keystore create -p
```

系统会提示输入密码。设置密码后，每次启动 Easysearch 时需要输入 keystore 密码。

> **注意**：如果 keystore 已存在，`create` 命令会提示是否覆盖。

## 添加设置

### 添加字符串设置

交互式输入（推荐，密码不会显示在终端历史中）：

```bash
bin/easysearch-keystore add s3.client.default.access_key
```

系统会提示输入值。

从标准输入读取（适合自动化脚本）：

```bash
echo "AKIAIOSFODNN7EXAMPLE" | bin/easysearch-keystore add --stdin s3.client.default.access_key
```

强制覆盖已有设置（不提示确认）：

```bash
bin/easysearch-keystore add --force s3.client.default.access_key
```

### 添加文件设置

将文件内容作为设置值存入 keystore：

```bash
bin/easysearch-keystore add-file my.certificate /path/to/cert.pem
```

## 查看已存储的设置

列出 keystore 中所有设置的名称（不显示值）：

```bash
bin/easysearch-keystore list
```

输出示例：

```
keystore.seed
s3.client.default.access_key
s3.client.default.secret_key
```

> **注意**：`keystore.seed` 是系统自动生成的随机种子，用于内部安全功能，请勿删除。

## 移除设置

```bash
bin/easysearch-keystore remove s3.client.default.access_key
```

## 修改 Keystore 密码

```bash
bin/easysearch-keystore passwd
```

系统会提示输入当前密码和新密码。

## 检查是否有密码保护

```bash
bin/easysearch-keystore has-passwd
```

如果 keystore 有密码保护，命令返回退出码 `0`；否则返回 `1`。

## 升级 Keystore 格式

```bash
bin/easysearch-keystore upgrade
```

将 keystore 升级到最新格式版本。通常在 Easysearch 版本升级后执行。

## 重新加载安全设置

修改 keystore 中的设置后，不需要重启整个 Easysearch 节点。可以使用 `_nodes/reload_secure_settings` API 在线重新加载：

```json
POST _nodes/reload_secure_settings
{
  "secure_settings_password": "keystore密码"
}
```

如果 keystore 没有设置密码，请求体可以为空：

```json
POST _nodes/reload_secure_settings
{}
```

响应会显示每个节点的重新加载状态：

```json
{
  "cluster_name": "my_cluster",
  "nodes": {
    "node_id_1": {
      "name": "node-1",
      "reload_exception": null
    }
  }
}
```

> **注意**：并非所有设置都支持在线重新加载。不支持的设置仍需要重启节点。

## 常见用法

### S3 快照仓库凭据

```bash
# 添加 S3 访问密钥
bin/easysearch-keystore add s3.client.default.access_key
bin/easysearch-keystore add s3.client.default.secret_key

# 如果使用临时凭据，还需添加 session token
bin/easysearch-keystore add s3.client.default.session_token

# 重新加载设置（无需重启）
curl -X POST "https://localhost:9200/_nodes/reload_secure_settings" \
  -H "Content-Type: application/json" \
  -d '{}' \
  --cacert config/certs/root-ca.pem \
  -u admin:密码
```

### Docker 环境中使用 Keystore

在 Docker 中，可以通过环境变量传入 keystore 密码：

```yaml
environment:
  - KEYSTORE_PASSWORD=my_keystore_password
```

或通过文件传入（更安全，配合 Docker Secrets）：

```yaml
environment:
  - KEYSTORE_PASSWORD_FILE=/run/secrets/keystore_password
secrets:
  keystore_password:
    file: ./keystore_password.txt
```

## 技术细节

| 属性 | 说明 |
|------|------|
| 加密算法 | AES/GCM/NoPadding |
| 密钥派生 | PBKDF2WithHmacSHA512（10,000 次迭代） |
| 密钥长度 | 128 位 |
| 盐长度 | 64 字节 |
| IV 长度 | 12 字节（96 位） |
| 设置名格式 | 字母、数字、下划线、连字符、点号（`[A-Za-z0-9_\-.]+`） |
| 文件权限 | `rw-rw----`（660） |

