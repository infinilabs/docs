---
title: "解除跟随者索引"
date: 0001-01-01
description: "在 Easysearch UI 中解除跟随者索引。"
summary: "解除跟随者索引 #  解除跟随者索引后，该索引将不再从主索引同步数据，变为独立索引。
操作步骤 #   进入管理页面：登录 Easysearch UI，左侧导航栏点击「主从复制」，默认进入「跟随者索引」标签页。 触发解除：找到目标跟随者索引（如 cloud_resource_usage_billing），点击「操作」列的更多按钮，选择「解除」。  确认执行：在弹出的提示框中核对索引名称，点击「确定」。  查看结果：操作完成后，目标索引从列表中移除，同步关系终止。  注意事项 #   解除仅断开同步，不会删除本地索引和远端主导索引的数据。 解除前请停止依赖该索引的业务，避免数据访问异常。 恢复同步需重新执行「新增跟随者索引」配置。  "
---


# 解除跟随者索引

解除跟随者索引后，该索引将不再从主索引同步数据，变为独立索引。

## 操作步骤

1. **进入管理页面**：登录 Easysearch UI，左侧导航栏点击「主从复制」，默认进入「跟随者索引」标签页。
2. **触发解除**：找到目标跟随者索引（如 `cloud_resource_usage_billing`），点击「操作」列的更多按钮，选择「解除」。

{{% load-img "/img/management/replication/remove-follower-index/image-1.png" %}}

3. **确认执行**：在弹出的提示框中核对索引名称，点击「确定」。

{{% load-img "/img/management/replication/remove-follower-index/image-2.png" %}}

4. **查看结果**：操作完成后，目标索引从列表中移除，同步关系终止。

{{% load-img "/img/management/replication/remove-follower-index/image-3.png" %}}

## 注意事项

- 解除仅断开同步，**不会删除**本地索引和远端主导索引的数据。
- 解除前请停止依赖该索引的业务，避免数据访问异常。
- 恢复同步需重新执行「新增跟随者索引」配置。

