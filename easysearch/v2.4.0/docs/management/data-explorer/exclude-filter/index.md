---
title: "排除过滤条件"
date: 0001-01-01
description: "在 Easysearch UI 数据探索中排除特定过滤条件。"
summary: "排除过滤条件 #  排除过滤条件可以将指定条件取反，从结果中排除匹配的数据。
操作步骤 #   唤起操作菜单：在数据探索页面，找到已生效的过滤条件标签（如 _id:nYsUlJ0Bc_MPNLBvYYQh），点击该标签，弹出操作菜单。 执行排除操作：在菜单中选择「排除结果」，系统将自动反转该过滤条件的匹配逻辑。  查看生效结果：  过滤标签将同步更新为 'NOT'_id:nYsUlJ0Bc_MPNLBvYYQh，标识当前为排除逻辑； 右侧数据列表将展示不满足该条件的所有文档记录，完成排除过滤。    "
---


# 排除过滤条件

排除过滤条件可以将指定条件取反，从结果中排除匹配的数据。

## 操作步骤

1. **唤起操作菜单**：在数据探索页面，找到已生效的过滤条件标签（如 `_id:nYsUlJ0Bc_MPNLBvYYQh`），点击该标签，弹出操作菜单。
2. **执行排除操作**：在菜单中选择「排除结果」，系统将自动反转该过滤条件的匹配逻辑。

{{% load-img "/img/management/data-explorer/exclude-filter/image-1.png" %}}

3. **查看生效结果**：
   - 过滤标签将同步更新为 `'NOT'_id:nYsUlJ0Bc_MPNLBvYYQh`，标识当前为排除逻辑；
   - 右侧数据列表将展示**不满足该条件**的所有文档记录，完成排除过滤。

{{% load-img "/img/management/data-explorer/exclude-filter/image-2.png" %}}

