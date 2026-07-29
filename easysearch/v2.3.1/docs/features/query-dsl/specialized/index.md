---
title: "专业查询"
date: 0001-01-01
description: "More_like_this、percolate、rank_feature、script_score 等高级查询。"
summary: "专业查询提供了用于特殊场景的高级功能，包括相似查询、反向查询、脚本评分等。
查询类型 #     查询 说明 使用场景      more_like_this 找出与指定文档相似的文档 &ldquo;相关阅读&rdquo;、推荐系统    percolate 反向查询（文档匹配规则） 规则引擎、警报、动态分类    rank_feature 排名特性查询 点击率、转化率等业务指标参与排名    script_score 脚本评分 自定义复杂评分逻辑    distance_feature 距离特性 地理位置距离参与评分    script 脚本查询（底层接口） 极其复杂的自定义逻辑    wrapper 包装查询（JSON 字符串） 动态查询构建    特点 #   灵活的评分控制：支持自定义评分逻辑 反向查询能力：percolate 提供规则引擎功能 高性能特性：rank_feature 针对特定场景优化 脚本支持：script、script_score 支持 Painless 脚本  推荐阅读顺序 #    more_like_this：相似查询  percolate：反向查询/规则引擎  rank_feature：排名特性  script_score：脚本评分  distance_feature：距离特性  script：脚本查询  wrapper：包装查询  常见用途 #     需求 推荐查询     &ldquo;相关阅读&quot;推荐  more_like_this   警报规则匹配  percolate   新闻自动分类  percolate   热度排序（考虑点击、转化等）  rank_feature   自定义复杂评分  script_score   地理距离参与评分  distance_feature 或 function_score    "
---


专业查询提供了用于特殊场景的高级功能，包括相似查询、反向查询、脚本评分等。

## 查询类型

| 查询 | 说明 | 使用场景 |
|------|------|---------|
| [more_like_this](./more-like-this.md) | 找出与指定文档相似的文档 | "相关阅读"、推荐系统 |
| [percolate](./percolate.md) | 反向查询（文档匹配规则） | 规则引擎、警报、动态分类 |
| [rank_feature](./rank-feature.md) | 排名特性查询 | 点击率、转化率等业务指标参与排名 |
| [script_score](./script-score.md) | 脚本评分 | 自定义复杂评分逻辑 |
| [distance_feature](./distance-feature.md) | 距离特性 | 地理位置距离参与评分 |
| [script](./script.md) | 脚本查询（底层接口） | 极其复杂的自定义逻辑 |
| [wrapper](./wrapper.md) | 包装查询（JSON 字符串） | 动态查询构建 |

## 特点

- **灵活的评分控制**：支持自定义评分逻辑
- **反向查询能力**：percolate 提供规则引擎功能
- **高性能特性**：rank_feature 针对特定场景优化
- **脚本支持**：script、script_score 支持 Painless 脚本

## 推荐阅读顺序

1. [more_like_this](./more-like-this.md)：相似查询
2. [percolate](./percolate.md)：反向查询/规则引擎
3. [rank_feature](./rank-feature.md)：排名特性
4. [script_score](./script-score.md)：脚本评分
5. [distance_feature](./distance-feature.md)：距离特性
6. [script](./script.md)：脚本查询
7. [wrapper](./wrapper.md)：包装查询

## 常见用途

| 需求 | 推荐查询 |
|------|---------|
| "相关阅读"推荐 | [more_like_this](./more-like-this.md) |
| 警报规则匹配 | [percolate](./percolate.md) |
| 新闻自动分类 | [percolate](./percolate.md) |
| 热度排序（考虑点击、转化等） | [rank_feature](./rank-feature.md) |
| 自定义复杂评分 | [script_score](./script-score.md) |
| 地理距离参与评分 | [distance_feature](./distance-feature.md) 或 function_score |
