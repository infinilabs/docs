---
title: "悬挂索引"
date: 0001-01-01
summary: "悬挂索引（Dangling Indices） #  什么是悬挂索引 #  悬挂索引是指存在于节点本地磁盘上，但不属于当前集群状态的索引。这种情况通常发生在：
 节点离线期间，集群中其他节点删除了某个索引 节点从一个集群迁移到另一个集群 快照恢复失败或中断 集群状态丢失后重建  当节点重新加入集群时，它本地存储的这些&quot;孤立&quot;索引分片就成为悬挂索引。
自动导入 #  可以通过集群设置控制是否自动导入悬挂索引：
# easysearch.yml gateway.auto_import_dangling_indices: false # 默认值：false  注意：自动导入默认关闭。在生产环境中，建议保持关闭，手动检查并决定是否导入，以避免意外引入过期或不需要的数据。
 API 操作 #  列出悬挂索引 #  GET /_dangling 响应示例：
{ &#34;dangling_indices&#34;: [ { &#34;index_name&#34;: &#34;my_old_index&#34;, &#34;index_uuid&#34;: &#34;r1eSJ3MoTheHQ2CvFoVrOg&#34;, &#34;creation_date_millis&#34;: 1609459200000, &#34;node_ids&#34;: [&#34;node-1&#34;] } ] } 导入悬挂索引 #  使用索引 UUID 将悬挂索引导入到集群中：
POST /_dangling/r1eSJ3MoTheHQ2CvFoVrOg?accept_data_loss=true accept_data_loss=true 参数是必须的，表示您已了解导入可能存在数据不一致的风险。
删除悬挂索引 #  不需要的悬挂索引可以直接删除：
DELETE /_dangling/r1eSJ3MoTheHQ2CvFoVrOg?accept_data_loss=true 操作建议 #     场景 建议操作     节点短暂离线后重新加入 检查索引内容后决定是否导入   集群迁移残留数据 确认数据已在新集群中恢复后删除   集群状态丢失重建 列出所有悬挂索引，逐一导入恢复   来源不明的悬挂索引 谨慎处理，建议先备份再决定    注意事项 #   导入悬挂索引不会自动恢复副本分片，需要等待集群自行分配 如果集群中已存在同名索引（但 UUID 不同），导入会失败 悬挂索引的映射和设置保持离线前的状态，可能与当前集群配置不兼容 建议在导入前通过 _dangling API 检查索引的创建时间，确认是否是期望的数据  "
---


# 悬挂索引（Dangling Indices）

## 什么是悬挂索引

悬挂索引是指存在于节点本地磁盘上，但不属于当前集群状态的索引。这种情况通常发生在：

- 节点离线期间，集群中其他节点删除了某个索引
- 节点从一个集群迁移到另一个集群
- 快照恢复失败或中断
- 集群状态丢失后重建

当节点重新加入集群时，它本地存储的这些"孤立"索引分片就成为悬挂索引。

## 自动导入

可以通过集群设置控制是否自动导入悬挂索引：

```yaml
# easysearch.yml
gateway.auto_import_dangling_indices: false  # 默认值：false
```

> **注意**：自动导入默认关闭。在生产环境中，建议保持关闭，手动检查并决定是否导入，以避免意外引入过期或不需要的数据。

## API 操作

### 列出悬挂索引

```
GET /_dangling
```

响应示例：

```json
{
  "dangling_indices": [
    {
      "index_name": "my_old_index",
      "index_uuid": "r1eSJ3MoTheHQ2CvFoVrOg",
      "creation_date_millis": 1609459200000,
      "node_ids": ["node-1"]
    }
  ]
}
```

### 导入悬挂索引

使用索引 UUID 将悬挂索引导入到集群中：

```
POST /_dangling/r1eSJ3MoTheHQ2CvFoVrOg?accept_data_loss=true
```

`accept_data_loss=true` 参数是必须的，表示您已了解导入可能存在数据不一致的风险。

### 删除悬挂索引

不需要的悬挂索引可以直接删除：

```
DELETE /_dangling/r1eSJ3MoTheHQ2CvFoVrOg?accept_data_loss=true
```

## 操作建议

| 场景 | 建议操作 |
|------|----------|
| 节点短暂离线后重新加入 | 检查索引内容后决定是否导入 |
| 集群迁移残留数据 | 确认数据已在新集群中恢复后删除 |
| 集群状态丢失重建 | 列出所有悬挂索引，逐一导入恢复 |
| 来源不明的悬挂索引 | 谨慎处理，建议先备份再决定 |

## 注意事项

- 导入悬挂索引不会自动恢复副本分片，需要等待集群自行分配
- 如果集群中已存在同名索引（但 UUID 不同），导入会失败
- 悬挂索引的映射和设置保持离线前的状态，可能与当前集群配置不兼容
- 建议在导入前通过 `_dangling` API 检查索引的创建时间，确认是否是期望的数据

