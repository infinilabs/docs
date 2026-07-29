---
title: "关联查询"
date: 0001-01-01
description: "Nested、has_child、has_parent、parent_id 关联查询。"
summary: "关联查询用于处理复杂的数据关系，包括嵌套文档（nested）和父子关系文档（parent-child）。
查询类型 #     查询 说明 适用场景      nested 嵌套文档查询 数组对象、一对多关系、保留数组元素的独立性    has_child 父文档查询（通过子文档条件） 大规模数据集、频繁更新子文档    has_parent 子文档查询（通过父文档条件） 返回子文档、通过父条件筛选    parent_id 按父文档 ID 查询子文档 直接查询特定父文档的所有子文档    嵌套（Nested）vs 父子（Parent-Child） #     特性 Nested Parent-Child     数据结构 单个文档中的对象数组 多个文档间的关系   查询性能 快 中等   索引性能 中等（文档膨胀） 快   更新成本 高（需重新索引整个文档） 低（独立更新）   数据规模 适合小到中等规模 适合大规模数据   适用场景 评论、标签、属性等 订单-商品、文章-评论等    推荐阅读顺序 #    nested：嵌套文档查询  has_child：父文档查询  has_parent：子文档查询  parent_id：ID 查询  常见用途 #     需求 使用查询     搜索：有好评的商品  nested（评论是商品的嵌套对象）   搜索：作者为 A 且评论分数 &gt; 4 的评论  has_parent（评论是子文档）   搜索：包含特定商品的订单  has_child（商品是订单的子文档）   直接查询某订单的所有商品  parent_id    "
---


关联查询用于处理复杂的数据关系，包括嵌套文档（nested）和父子关系文档（parent-child）。

## 查询类型

| 查询 | 说明 | 适用场景 |
|------|------|---------|
| [nested](./nested.md) | 嵌套文档查询 | 数组对象、一对多关系、保留数组元素的独立性 |
| [has_child](./has-child.md) | 父文档查询（通过子文档条件） | 大规模数据集、频繁更新子文档 |
| [has_parent](./has-parent.md) | 子文档查询（通过父文档条件） | 返回子文档、通过父条件筛选 |
| [parent_id](./parent-id.md) | 按父文档 ID 查询子文档 | 直接查询特定父文档的所有子文档 |

## 嵌套（Nested）vs 父子（Parent-Child）

| 特性 | Nested | Parent-Child |
|------|--------|-------------|
| 数据结构 | 单个文档中的对象数组 | 多个文档间的关系 |
| 查询性能 | 快 | 中等 |
| 索引性能 | 中等（文档膨胀） | 快 |
| 更新成本 | 高（需重新索引整个文档） | 低（独立更新） |
| 数据规模 | 适合小到中等规模 | 适合大规模数据 |
| 适用场景 | 评论、标签、属性等 | 订单-商品、文章-评论等 |

## 推荐阅读顺序

1. [nested](./nested.md)：嵌套文档查询
2. [has_child](./has-child.md)：父文档查询
3. [has_parent](./has-parent.md)：子文档查询
4. [parent_id](./parent-id.md)：ID 查询

## 常见用途

| 需求 | 使用查询 |
|------|---------|
| 搜索：有好评的商品 | [nested](./nested.md)（评论是商品的嵌套对象） |
| 搜索：作者为 A 且评论分数 > 4 的评论 | [has_parent](./has-parent.md)（评论是子文档） |
| 搜索：包含特定商品的订单 | [has_child](./has-child.md)（商品是订单的子文档） |
| 直接查询某订单的所有商品 | [parent_id](./parent-id.md) |
