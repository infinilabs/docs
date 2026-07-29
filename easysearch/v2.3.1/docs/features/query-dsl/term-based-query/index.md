---
title: "精确查询"
date: 0001-01-01
description: "Term、terms、range、prefix、wildcard、regexp、fuzzy 等精确查询类型。"
summary: "精确查询用于不进行分词的精确值匹配。这类查询直接与倒排索引中的 keyword 字段匹配，不经过分析过程，通常不参与相关性评分。
精确查询与全文检索的区别 #  精确查询与全文检索的核心差异在于：精确查询搜索文档中的确切词项，而全文检索会先对查询字符串进行分词分析。
    精确查询 全文检索     描述 回答&quot;哪些文档匹配&quot; 回答&quot;文档与查询的匹配程度&quot;   分析器 搜索词不被分析，按字面值匹配 搜索词与文档字段使用相同分析器   相关性 不根据相关性分数排序（所有匹配文档评分相同） 计算相关性分数并按分数降序排序   用例 精确值匹配（数字、日期、标签等 keyword 字段） 文本字段的全文搜索     注意：精确查询不适合搜索分词后的 text 字段，通常会产生意想不到的结果（无匹配）。在处理文本数据时，仅将精确查询用于映射为 keyword 的字段。
 示例：为什么 term 在 text 字段上搜不到？ #  使用精确查询搜索 text_entry（text 字段）中的 &ldquo;To be, or not to be&rdquo;：
GET shakespeare/_search { &#34;query&#34;: { &#34;term&#34;: { &#34;text_entry&#34;: &#34;To be, or not to be&#34; } } } 结果为 0 条匹配，因为倒排索引中存储的是分词后的小写词项（to、be、or、not），而 term 查询会将整个短语作为一个完整的词项去匹配。"
---


精确查询用于不进行分词的精确值匹配。这类查询直接与倒排索引中的 keyword 字段匹配，不经过分析过程，通常不参与相关性评分。

## 精确查询与全文检索的区别

精确查询与全文检索的核心差异在于：精确查询搜索文档中的**确切词项**，而全文检索会先对查询字符串进行**分词分析**。

|        | 精确查询 | 全文检索 |
| ------ | -------- | -------- |
| 描述   | 回答"哪些文档匹配" | 回答"文档与查询的匹配程度" |
| 分析器 | 搜索词**不被分析**，按字面值匹配 | 搜索词与文档字段使用**相同分析器** |
| 相关性 | 不根据相关性分数排序（所有匹配文档评分相同） | 计算相关性分数并按分数降序排序 |
| 用例   | 精确值匹配（数字、日期、标签等 keyword 字段） | 文本字段的全文搜索 |

> **注意**：精确查询不适合搜索分词后的 `text` 字段，通常会产生意想不到的结果（无匹配）。在处理文本数据时，仅将精确查询用于映射为 `keyword` 的字段。

### 示例：为什么 term 在 text 字段上搜不到？

使用精确查询搜索 `text_entry`（text 字段）中的 "To be, or not to be"：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "text_entry": "To be, or not to be"
    }
  }
}
```

结果为 0 条匹配，因为倒排索引中存储的是分词后的小写词项（`to`、`be`、`or`、`not`），而 `term` 查询会将整个短语作为一个完整的词项去匹配。

改用 `match` 全文查询，则可以正确搜索到结果——搜索词会被分词后与倒排索引匹配，并按相关性排序。

### 示例：在 keyword 字段上使用精确查询

如果要在 `speaker`（keyword 字段）中搜索精确值 "HAMLET"：

```json
GET shakespeare/_search
{
  "query": {
    "term": {
      "speaker": "HAMLET"
    }
  }
}
```

精确匹配返回 1582 条结果。注意必须大小写完全一致——搜索 "Hamlet" 将无匹配，因为 keyword 字段以原文形式存储。

## 查询类型

| 查询 | 说明 | 使用场景 |
|------|------|---------|
| [term](./term.md) | 精确匹配单个值 | 精确搜索（ID、状态码、分类等） |
| [terms](./terms.md) | 精确匹配多个值，任一匹配即可 | 多选筛选 |
| [range](./range.md) | 范围查询（数值、日期） | 日期范围、价格范围、年龄范围 |
| [prefix](./prefix.md) | 前缀匹配（不分词） | 自动补全、标签前缀匹配 |
| [wildcard](./wildcard.md) | 通配符匹配（? 单字符，* 多字符） | 灵活模式匹配 |
| [regexp](./regexp.md) | 正则表达式匹配（Lucene 语法） | 复杂模式匹配 |
| [fuzzy](./fuzzy.md) | 模糊匹配（编辑距离） | 拼写纠正、相似值查询 |
| [exists](./exists.md) | 检查字段是否存在 | 非空检查 |
| [ids](./ids.md) | 按文档 ID 查询 | 直接查询特定文档 |
| [terms_set](./terms-set.md) | 动态词项匹配 | 按最少匹配数筛选 |

## 特点

- **不分词**：精确匹配，不进行分词分析
- **高效**：通常不计算相关性评分，返回结果快
- **在 filter 上下文中使用**：常用于 bool 查询的 filter 子句
- **缓存友好**：适合作为过滤条件被缓存

## 推荐阅读顺序

1. [term](./term.md)：单值精确匹配
2. [terms](./terms.md)：多值精确匹配
3. [range](./range.md)：范围查询
4. [prefix](./prefix.md)：前缀匹配
5. [wildcard](./wildcard.md)：通配符
6. [regexp](./regexp.md)：正则表达式
7. [fuzzy](./fuzzy.md)：模糊匹配
8. [exists](./exists.md)：存在性检查
9. [ids](./ids.md)：ID 查询
10. [terms_set](./terms-set.md)：动态匹配

## 常见用途

| 场景 | 推荐查询 |
|------|---------|
| 精确查找一个值 | [term](./term.md) |
| 精确查找多个值之一 | [terms](./terms.md) |
| 日期范围筛选 | [range](./range.md) |
| 价格范围筛选 | [range](./range.md) |
| 自动补全前缀 | [prefix](./prefix.md) |
| 灵活的通配符匹配 | [wildcard](./wildcard.md) |
| 拼写纠正 | [fuzzy](./fuzzy.md) |
| 检查字段是否为空 | [exists](./exists.md) |
