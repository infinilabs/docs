---
title: "Span 查询"
date: 0001-01-01
description: "Span 查询族：精细的文本级查询，用于高级搜索场景。"
summary: "Span 查询提供了文本级的精细控制，允许对词项位置、距离、顺序等进行精确控制。Span 查询常用于法律文档搜索、学术文献检索、高级信息提取等场景。
特点 #   词项级精细控制：可以精确控制词项间的距离、顺序、位置 位置感知：基于词项在文本中的位置进行匹配 高性能：内置缓存和优化，适合大规模查询 可组合：支持嵌套和组合多个 span 查询  Span 查询类型 #  基础 Span 查询 #     查询 说明      span_term 单个词项的位置查询    span_multi_term 多个词项的位置查询（支持通配符、前缀、范围）    组合 Span 查询 #     查询 说明      span_near 词项距离约束（距离 &lt;= slop）    span_or 多个 span 查询的或关系    span_not 排除特定词项位置    位置约束 #     查询 说明      span_first 限制匹配位置在前 N 个词项内    span_containing 匹配包含另一个 span 匹配的文本    span_within 匹配被另一个 span 匹配包含的文本    高级 #     查询 说明      span_field_masking 跨字段查询（在一个字段中应用另一个字段的匹配）    推荐阅读顺序 #    span_term：基础的 span 查询  span_near：距离约束  span_or：或逻辑  span_not：否定  span_first：位置限制  span_containing / span_within：包含关系  span_field_masking：跨字段应用  常见用途 #     场景 使用 Span 查询     找出词项 A 和 B 在 5 词内出现的文档  span_near with slop=5   找出不包含某词的匹配  span_not   找出匹配出现在前 100 个词内  span_first with end=100   找出包含特定短语的文档  span_near    "
---


Span 查询提供了文本级的精细控制，允许对词项位置、距离、顺序等进行精确控制。Span 查询常用于法律文档搜索、学术文献检索、高级信息提取等场景。

## 特点

- **词项级精细控制**：可以精确控制词项间的距离、顺序、位置
- **位置感知**：基于词项在文本中的位置进行匹配
- **高性能**：内置缓存和优化，适合大规模查询
- **可组合**：支持嵌套和组合多个 span 查询

## Span 查询类型

### 基础 Span 查询

| 查询 | 说明 |
|------|------|
| [span_term](./span-term.md) | 单个词项的位置查询 |
| [span_multi_term](./span-multi-term.md) | 多个词项的位置查询（支持通配符、前缀、范围） |

### 组合 Span 查询

| 查询 | 说明 |
|------|------|
| [span_near](./span-near.md) | 词项距离约束（距离 <= slop） |
| [span_or](./span-or.md) | 多个 span 查询的或关系 |
| [span_not](./span-not.md) | 排除特定词项位置 |

### 位置约束

| 查询 | 说明 |
|------|------|
| [span_first](./span-first.md) | 限制匹配位置在前 N 个词项内 |
| [span_containing](./span-containing.md) | 匹配包含另一个 span 匹配的文本 |
| [span_within](./span-within.md) | 匹配被另一个 span 匹配包含的文本 |

### 高级

| 查询 | 说明 |
|------|------|
| [span_field_masking](./span-field-masking.md) | 跨字段查询（在一个字段中应用另一个字段的匹配） |

## 推荐阅读顺序

1. [span_term](./span-term.md)：基础的 span 查询
2. [span_near](./span-near.md)：距离约束
3. [span_or](./span-or.md)：或逻辑
4. [span_not](./span-not.md)：否定
5. [span_first](./span-first.md)：位置限制
6. [span_containing / span_within](./span-containing.md)：包含关系
7. [span_field_masking](./span-field-masking.md)：跨字段应用

## 常见用途

| 场景 | 使用 Span 查询 |
|------|--------------|
| 找出词项 A 和 B 在 5 词内出现的文档 | [span_near](./span-near.md) with slop=5 |
| 找出不包含某词的匹配 | [span_not](./span-not.md) |
| 找出匹配出现在前 100 个词内 | [span_first](./span-first.md) with end=100 |
| 找出包含特定短语的文档 | [span_near](./span-near.md) |
