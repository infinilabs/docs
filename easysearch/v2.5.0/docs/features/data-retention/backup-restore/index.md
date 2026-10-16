---
title: "备份与恢复"
date: 0001-01-01
description: "快照仓库配置、备份策略、增量备份原理与跨集群恢复实战。"
summary: "备份与恢复 #  副本提供了高可用性，但无法防御灾难性故障（如误删除、数据损坏、机房级故障）。快照（Snapshot） 是 Easysearch 的备份机制，提供完整的数据保护能力。
 快照机制概述 #  核心特点 #   增量备份：首次快照是全量备份，后续快照只保存变化的数据 不阻塞写入：快照过程不会阻止正常的索引和搜索操作 支持多种存储：共享文件系统、S3、HDFS、Azure Blob 等 灵活恢复：可恢复到同一集群或不同集群，支持重命名  工作流程 #  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │ 创建仓库 │───▶│ 执行快照 │───▶│ 恢复数据 │ └─────────────┘ └─────────────┘ └─────────────┘ │ │ │ ▼ ▼ ▼ 配置存储位置 选择索引范围 选择目标索引 设置访问权限 等待快照完成 可重命名恢复  创建快照仓库 #  快照需要先配置一个仓库（Repository），仓库定义了快照的存储位置。
共享文件系统仓库 #  前提条件：
 所有节点都能访问同一个共享目录（NFS、GlusterFS 等） 在 easysearch.yml 中配置允许的路径：  path.repo: [&#34;/mount/backups&#34;, &#34;/mount/snapshots&#34;] 创建仓库："
---


# 备份与恢复

副本提供了高可用性，但无法防御灾难性故障（如误删除、数据损坏、机房级故障）。**快照（Snapshot）** 是 Easysearch 的备份机制，提供完整的数据保护能力。

---

## 快照机制概述

### 核心特点

- **增量备份**：首次快照是全量备份，后续快照只保存变化的数据
- **不阻塞写入**：快照过程不会阻止正常的索引和搜索操作
- **支持多种存储**：共享文件系统、S3、HDFS、Azure Blob 等
- **灵活恢复**：可恢复到同一集群或不同集群，支持重命名

### 工作流程

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  创建仓库   │───▶│  执行快照   │───▶│  恢复数据   │
└─────────────┘    └─────────────┘    └─────────────┘
       │                  │                  │
       ▼                  ▼                  ▼
  配置存储位置      选择索引范围        选择目标索引
  设置访问权限      等待快照完成        可重命名恢复
```

---

## 创建快照仓库

快照需要先配置一个仓库（Repository），仓库定义了快照的存储位置。

### 共享文件系统仓库

**前提条件**：
1. 所有节点都能访问同一个共享目录（NFS、GlusterFS 等）
2. 在 `easysearch.yml` 中配置允许的路径：

```yaml
path.repo: ["/mount/backups", "/mount/snapshots"]
```

**创建仓库**：

```json
PUT _snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/mount/backups/easysearch",
    "compress": true,
    "max_snapshot_bytes_per_sec": "50mb",
    "max_restore_bytes_per_sec": "50mb"
  }
}
```

**参数说明**：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `location` | 必填 | 快照存储路径 |
| `compress` | `true` | 是否压缩元数据文件 |
| `max_snapshot_bytes_per_sec` | `40mb` | 快照写入限速 |
| `max_restore_bytes_per_sec` | `40mb` | 恢复读取限速 |

**查询参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| `verify` | `boolean` | 创建后是否自动验证仓库（默认自动验证） |
| `master_timeout` | `time` | 连接主节点的超时时间 |
| `timeout` | `time` | 操作超时时间 |

### S3 兼容存储仓库

1. 将 AWS 访问密钥添加到 Easysearch 密钥库：

```bash
sudo ./bin/easysearch-keystore add s3.client.default.access_key
sudo ./bin/easysearch-keystore add s3.client.default.secret_key
```

2. （可选）如果使用临时凭据，添加会话令牌：

```bash
sudo ./bin/easysearch-keystore add s3.client.default.session_token
```

3. （可选）如果通过代理连接，添加代理凭据：

```bash
sudo ./bin/easysearch-keystore add s3.client.default.proxy.username
sudo ./bin/easysearch-keystore add s3.client.default.proxy.password
```

4. （可选）将其他 S3 客户端设置添加到 `easysearch.yml`：

```yml
s3.client.default.endpoint: s3.amazonaws.com
s3.client.default.protocol: https
s3.client.default.proxy.host: my-proxy-host
s3.client.default.proxy.port: 8080
s3.client.default.read_timeout: 50s
s3.client.default.max_retries: 3
s3.client.default.use_throttle_retries: true
s3.client.default.path_style_access: false
s3.client.default.disable_chunked_encoding: false
```

5. 如果修改了 `easysearch.yml`，需要重启节点。否则只需重新加载安全设置：

```
POST _nodes/reload_secure_settings
```

6. 确保 S3 bucket 存在，并具备访问权限。以下 IAM 策略示例：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": ["s3:*"],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::your-bucket",
        "arn:aws:s3:::your-bucket/*"
      ]
    }
  ]
}
```

7. 注册仓库：

```json
PUT _snapshot/s3_backup
{
  "type": "s3",
  "settings": {
    "bucket": "my-backup-bucket",
    "base_path": "easysearch/snapshots",
    "compress": true
  }
}
```

**S3 仓库参数**：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `bucket` | 必填 | S3 bucket 名称 |
| `base_path` | 根目录 | bucket 中的存储路径 |
| `client` | `default` | 客户端配置名称，对应 `s3.client.<name>` 设置 |
| `compress` | `false` | 是否压缩元数据文件 |
| `chunk_size` | `1gb` | 文件分块大小 |
| `buffer_size` | min(100MB, 5% 堆) | 分块上传的缓冲区大小（5MB ～ 5GB） |
| `server_side_encryption` | `false` | 是否启用 S3 服务端 AES-256 加密 |
| `canned_acl` | `private` | S3 预定义 ACL |
| `storage_class` | `standard` | S3 存储类型（不要使用 `glacier` 和 `deep_archive`） |
| `max_snapshot_bytes_per_sec` | `40mb` | 快照写入限速 |
| `max_restore_bytes_per_sec` | `40mb` | 恢复读取限速 |
| `readonly` | `false` | 是否只读 |

### URL 仓库（只读）

URL 仓库是一种**只读**的快照仓库类型，用于从远程 HTTP/HTTPS 地址恢复快照。它通常用于：

- 从其他集群共享的快照中恢复数据
- 从公开发布的快照仓库中恢复数据
- 将共享文件系统仓库以 HTTP 方式暴露供远程集群访问

**前提条件**：在 `easysearch.yml` 中配置允许的 URL 地址：

```yaml
repositories.url.allowed_urls: ["http://backup-server.example.com/snapshots/*", "https://cdn.example.com/es-snapshots/*"]
```

> 支持 `*`（匹配任意字符）和 `**`（匹配多层路径）通配符。`file://` 协议也受支持，但需要在 `path.repo` 中注册路径。

**注册 URL 仓库**：

```json
PUT _snapshot/remote_backup
{
  "type": "url",
  "settings": {
    "url": "http://backup-server.example.com/snapshots/easysearch/"
  }
}
```

**URL 仓库参数**：

| 参数 | 说明 |
|------|------|
| `url` | 快照数据的根 URL（必填），支持 `http`、`https`、`ftp`、`file` 和 `jar` 协议 |

> **使用场景**：集群 A 使用共享文件系统仓库创建快照，通过 Web 服务器（如 Nginx）将快照目录发布为 HTTP 服务，集群 B 使用 URL 仓库从该地址恢复数据。

### 验证仓库配置

```json
POST _snapshot/my_backup/_verify
```

**成功响应**：
```json
{
  "nodes": {
    "node-1": { "name": "node-1" },
    "node-2": { "name": "node-2" }
  }
}
```

### 查看所有仓库

```json
GET _snapshot/_all
```

---

## 创建快照

### 快照所有索引

```json
PUT _snapshot/my_backup/snapshot_20260211
{
  "indices": "*",
  "ignore_unavailable": true,
  "include_global_state": true
}
```

**参数说明**：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `indices` | 所有索引 | 要备份的索引（支持通配符） |
| `ignore_unavailable` | `false` | 是否忽略不存在的索引 |
| `include_global_state` | `true` | 是否包含集群状态（模板、设置等） |

### 快照指定索引

```json
PUT _snapshot/my_backup/snapshot_logs_20260211
{
  "indices": "logs-2026.02.*",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

### 等待快照完成

默认情况下，快照请求立即返回，快照在后台执行。如需等待完成：

```json
PUT _snapshot/my_backup/snapshot_20260211?wait_for_completion=true
```

> 注意：大型快照可能需要较长时间，建议在脚本中使用轮询方式监控进度。

---

## 监控快照进度

### 查看快照状态

```json
GET _snapshot/my_backup/snapshot_20260211
```

**响应示例**：
```json
{
  "snapshots": [
    {
      "snapshot": "snapshot_20260211",
      "uuid": "abc123...",
      "state": "SUCCESS",
      "start_time": "2026-02-11T02:00:00.000Z",
      "end_time": "2026-02-11T02:15:30.000Z",
      "duration_in_millis": 930000,
      "indices": ["logs-2026.02.10", "logs-2026.02.11"],
      "shards": {
        "total": 10,
        "failed": 0,
        "successful": 10
      }
    }
  ]
}
```

**快照状态**：

| 状态 | 说明 |
|------|------|
| `IN_PROGRESS` | 快照正在进行 |
| `SUCCESS` | 快照成功完成 |
| `FAILED` | 快照失败 |
| `PARTIAL` | 部分分片失败，但快照已创建（仅在 `partial: true` 时出现） |
| `INCOMPATIBLE` | 快照与当前 Easysearch 版本不兼容 |

### 查看进行中快照的详细进度

```json
GET _snapshot/my_backup/snapshot_20260211/_status
```

**响应示例**：
```json
{
  "snapshots": [
    {
      "snapshot": "snapshot_20260211",
      "state": "IN_PROGRESS",
      "shards_stats": {
        "initializing": 0,
        "started": 2,
        "finalizing": 1,
        "done": 7,
        "failed": 0,
        "total": 10
      },
      "indices": {
        "logs-2026.02.11": {
          "shards_stats": {
            "done": 3,
            "total": 5
          },
          "shards": {
            "0": {
              "stage": "DONE",
              "stats": {
                "total": {
                  "file_count": 15,
                  "size_in_bytes": 1073741824
                },
                "processed": {
                  "file_count": 15,
                  "size_in_bytes": 1073741824
                }
              }
            }
          }
        }
      }
    }
  ]
}
```

### 列出所有快照

```json
GET _snapshot/my_backup/_all
```

---

## 恢复快照

### 恢复所有索引

```json
POST _snapshot/my_backup/snapshot_20260211/_restore
```

### 恢复指定索引

```json
POST _snapshot/my_backup/snapshot_20260211/_restore
{
  "indices": "logs-2026.02.10",
  "ignore_unavailable": true
}
```

### 恢复并重命名索引

当需要恢复到已存在的集群，或者想要对比新旧数据时：

```json
POST _snapshot/my_backup/snapshot_20260211/_restore
{
  "indices": "logs-2026.02.10",
  "rename_pattern": "(.+)",
  "rename_replacement": "restored_$1"
}
```

**结果**：`logs-2026.02.10` 恢复为 `restored_logs-2026.02.10`

### 恢复到不同的索引设置

```json
POST _snapshot/my_backup/snapshot_20260211/_restore
{
  "indices": "logs-2026.02.10",
  "index_settings": {
    "index.number_of_replicas": 0
  },
  "ignore_index_settings": [
    "index.refresh_interval"
  ]
}
```

> 技巧：恢复时先设置 `number_of_replicas: 0`，恢复完成后再调整为期望值，可以加快恢复速度。

### 恢复请求参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `indices` | 所有索引 | 要恢复的索引（支持通配符和 `-` 排除） |
| `ignore_unavailable` | `false` | 是否忽略不存在的索引 |
| `include_global_state` | `false` | 是否恢复集群状态 |
| `include_aliases` | `true` | 是否恢复别名及关联索引 |
| `partial` | `false` | 是否允许恢复部分快照 |
| `rename_pattern` | — | 匹配索引名的正则表达式，配合 `rename_replacement` 使用 |
| `rename_replacement` | — | 替换模式。`$0` 代表完整匹配，`$1` 代表第一个捕获组 |
| `index_settings` | — | 恢复时覆盖的索引设置 |
| `ignore_index_settings` | — | 恢复时忽略的索引设置（使用集群默认值） |

### 等待恢复完成

```json
POST _snapshot/my_backup/snapshot_20260211/_restore?wait_for_completion=true
```

---

## 监控恢复进度

### 查看恢复状态

```json
GET _cat/recovery?v&active_only=true
```

**输出示例**：
```
index                 shard time   type       stage source_node target_node files_percent bytes_percent
logs-2026.02.10       0     2m     snapshot   index n/a         node-1      85.0%         72.3%
logs-2026.02.10       1     1m30s  snapshot   index n/a         node-2      100.0%        100.0%
```

### 查看特定索引的恢复进度

```json
GET logs-2026.02.10/_recovery
```

---

## 删除快照

使用 API 删除快照（**不要手动删除文件**）：

```json
DELETE _snapshot/my_backup/snapshot_20260210
```

> **重要**：快照是增量的，手动删除文件可能损坏其他快照。始终使用 API 删除。

### 取消正在进行的快照

```json
DELETE _snapshot/my_backup/snapshot_in_progress
```

---

## 克隆快照

在同一个仓库内，将一个快照中的部分索引克隆到一个新的快照中，无需重新读取原始数据：

```json
PUT _snapshot/my_backup/snapshot_20260211/_clone/snapshot_logs_only
{
  "indices": "logs-*"
}
```

> **使用场景**：需要保留特定索引的快照副本，或在清理旧快照前单独保存关键数据。

---

## 清理仓库

删除快照后，仓库中可能残留无用的数据文件。使用清理 API 回收这些孤立数据：

```json
POST _snapshot/my_backup/_cleanup
```

**响应示例**：
```json
{
  "results": {
    "deleted_bytes": 1073741824,
    "deleted_blobs": 42
  }
}
```

> **建议**：在批量删除快照后执行一次清理操作。

---

## 删除仓库

```json
DELETE _snapshot/my_backup
```

> **注意**：删除仓库只会取消注册，不会删除存储中的快照数据。如需清除存储空间，需要手动删除底层存储中的文件。

---

## 备份策略建议

### 备份频率

| 数据类型 | 建议频率 | 保留周期 |
|----------|----------|----------|
| 业务核心数据 | 每日 | 30-90 天 |
| 日志/指标 | 每日或每周 | 7-30 天 |
| 配置/模板 | 变更时 | 永久 |

### 命名规范

推荐使用包含日期和类型的命名：

```
snapshot_全量_20260211
snapshot_logs_20260211
snapshot_hourly_2026021110
```

### 自动化备份脚本

```bash
#!/bin/bash
# daily_backup.sh - 每日自动备份脚本

ES_HOST="http://localhost:9200"
REPO="my_backup"
DATE=$(date +%Y%m%d)
SNAPSHOT_NAME="snapshot_daily_${DATE}"

# 创建快照
curl -X PUT "${ES_HOST}/_snapshot/${REPO}/${SNAPSHOT_NAME}?wait_for_completion=true" \
  -H 'Content-Type: application/json' \
  -d '{
    "indices": "*",
    "ignore_unavailable": true,
    "include_global_state": true
  }'

# 检查结果
STATUS=$(curl -s "${ES_HOST}/_snapshot/${REPO}/${SNAPSHOT_NAME}" | jq -r '.snapshots[0].state')
if [ "$STATUS" != "SUCCESS" ]; then
  echo "Backup failed: $STATUS"
  # 发送告警...
  exit 1
fi

echo "Backup completed: ${SNAPSHOT_NAME}"

# 清理旧快照（保留最近 30 天）
CUTOFF_DATE=$(date -d "30 days ago" +%Y%m%d)
for snapshot in $(curl -s "${ES_HOST}/_snapshot/${REPO}/_all" | jq -r '.snapshots[].snapshot'); do
  SNAP_DATE=$(echo $snapshot | grep -oP '\d{8}')
  if [[ -n "$SNAP_DATE" && "$SNAP_DATE" < "$CUTOFF_DATE" ]]; then
    curl -X DELETE "${ES_HOST}/_snapshot/${REPO}/${snapshot}"
    echo "Deleted old snapshot: ${snapshot}"
  fi
done
```

### 使用 SLM 自动化（推荐）

Easysearch 的快照生命周期管理（SLM）可以自动执行备份策略：

```json
PUT _slm/policy/daily-snapshots
{
  "description": "每日自动备份",
  "creation": {
    "schedule": {
      "cron": {
        "expression": "0 30 2 * * ?",
        "timezone": "Asia/Shanghai"
      }
    }
  },
  "snapshot_config": {
    "date_expression": "<daily-snap-{now/d}>",
    "repository": "my_backup",
    "indices": "*",
    "include_global_state": true
  },
  "deletion": {
    "condition": {
      "max_age": "30d",
      "min_count": 5,
      "max_count": 50
    }
  }
}
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `creation.schedule.cron.expression` | Cron 表达式（每天凌晨 2:30） |
| `snapshot_config.date_expression` | 快照名称模板（支持日期变量） |
| `snapshot_config.repository` | 快照仓库名称 |
| `deletion.condition.max_age` | 快照过期时间 |
| `deletion.condition.min_count` | 最少保留快照数 |
| `deletion.condition.max_count` | 最多保留快照数 |

**手动执行策略**：

```json
POST _slm/policy/daily-snapshots/_execute
```

**查看策略状态**：

```json
GET _slm/policy/daily-snapshots
```

---

## 安全模块注意事项

如果启用了安全模块，快照操作有额外限制：

- 用户必须具备内置 `manage_snapshots` 角色
- 无法恢复包含全局状态或 `.security` 索引的快照

如果快照包含全局状态，恢复时必须排除：

```json
POST _snapshot/my_backup/snapshot_20260211/_restore
{
  "indices": "-.security",
  "include_global_state": false
}
```

如果确实需要恢复 `.security` 索引，必须在请求中包含管理员证书：

```bash
curl -k --cert ./admin.pem --key ./admin-key.pem \
  -XPOST 'https://localhost:9200/_snapshot/my_backup/snapshot_20260211/_restore?pretty'
```

---

## 版本兼容性

快照只能**向前兼容一个主要版本**。例如：
- 在 1.x 集群上创建的快照可以在 1.x 或 2.x 集群上恢复
- 但不能在 6.x 集群上恢复

如果有一个旧版本快照，可以尝试恢复到中间版本集群，重新索引后再创建新快照逐步迁移。

---

## 跨集群恢复

### 场景：恢复到新集群

1. **在新集群注册相同的仓库**（指向同一存储位置）：

```json
PUT _snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/mount/backups/easysearch",
    "readonly": true
  }
}
```

> 注意：如果只需要恢复，建议设置 `readonly: true` 防止误操作。

2. **查看可用快照**：

```json
GET _snapshot/my_backup/_all
```

3. **执行恢复**：

```json
POST _snapshot/my_backup/snapshot_20260211/_restore
{
  "indices": "logs-*",
  "ignore_unavailable": true
}
```

### 场景：灾难恢复演练

定期进行恢复演练，验证备份的有效性：

```json
# 1. 恢复到临时索引
POST _snapshot/my_backup/snapshot_20260211/_restore
{
  "indices": "critical-data",
  "rename_pattern": "(.+)",
  "rename_replacement": "dr_test_$1"
}

# 2. 验证数据
GET dr_test_critical-data/_count
GET dr_test_critical-data/_search?size=10

# 3. 清理测试数据
DELETE dr_test_critical-data
```

---

## 常见问题

### 快照失败：磁盘空间不足

**症状**：快照状态为 `FAILED`，错误信息包含 `disk space`

**解决方案**：
1. 清理旧快照释放空间
2. 扩展仓库存储容量
3. 分批备份大索引

### 恢复失败：索引已存在

**症状**：恢复时报错 `index already exists`

**解决方案**：
```json
# 方案 1：先删除目标索引
DELETE logs-2026.02.10

# 方案 2：使用重命名恢复
POST _snapshot/my_backup/snapshot_20260211/_restore
{
  "indices": "logs-2026.02.10",
  "rename_pattern": "(.+)",
  "rename_replacement": "restored_$1"
}
```

### 恢复很慢

**优化建议**：
1. 恢复时设置 `number_of_replicas: 0`，完成后再调整
2. 增加 `max_restore_bytes_per_sec` 限速值
3. 确保网络带宽充足

```json
POST _snapshot/my_backup/snapshot_20260211/_restore
{
  "indices": "big-index",
  "index_settings": {
    "index.number_of_replicas": 0
  }
}

# 恢复完成后
PUT big-index/_settings
{
  "index.number_of_replicas": 1
}
```

---

## API 参考汇总

### 仓库管理

| 操作 | 方法 | 端点 |
|------|------|------|
| 创建/更新仓库 | `PUT` / `POST` | `/_snapshot/{repository}` |
| 查看仓库 | `GET` | `/_snapshot` 或 `/_snapshot/{repository}` |
| 删除仓库 | `DELETE` | `/_snapshot/{repository}` |
| 验证仓库 | `POST` | `/_snapshot/{repository}/_verify` |
| 清理仓库 | `POST` | `/_snapshot/{repository}/_cleanup` |

### 快照操作

| 操作 | 方法 | 端点 |
|------|------|------|
| 创建快照 | `PUT` / `POST` | `/_snapshot/{repository}/{snapshot}` |
| 查看快照 | `GET` | `/_snapshot/{repository}/{snapshot}` |
| 快照状态 | `GET` | `/_snapshot/_status` 或 `/_snapshot/{repository}/{snapshot}/_status` |
| 克隆快照 | `PUT` | `/_snapshot/{repository}/{snapshot}/_clone/{target_snapshot}` |
| 删除快照 | `DELETE` | `/_snapshot/{repository}/{snapshot}` |
| 恢复快照 | `POST` | `/_snapshot/{repository}/{snapshot}/_restore` |

### 快照生命周期管理

| 操作 | 方法 | 端点 |
|------|------|------|
| 创建/更新策略 | `PUT` | `/_slm/policy/{policy_id}` |
| 查看策略 | `GET` | `/_slm/policy/{policy_id}` |
| 执行策略 | `POST` | `/_slm/policy/{policy_id}/_execute` |
| 删除策略 | `DELETE` | `/_slm/policy/{policy_id}` |

**最佳实践**：
- 定期备份，使用 SLM 自动化
- 定期验证，进行恢复演练
- 异地存储，防止机房级故障
- 监控告警，及时发现备份失败

**下一步阅读**：
- [索引生命周期管理]({{< relref "/docs/features/data-retention/ilm" >}})：自动化索引生命周期
- [SLM 快照生命周期]({{< relref "/docs/features/data-retention/slm" >}})：自动化快照策略
- [可搜索快照]({{< relref "/docs/features/data-retention/searchable-snapshot" >}})：从快照直接搜索
- [数据生命周期]({{< relref "/docs/features/data-retention/lifecycle" >}})：完整的数据保留策略

