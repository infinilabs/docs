---
title: "映射基础"
date: 0001-01-01
description: "核心映射结构、动态映射与常见字段类型。"
summary: "映射基础 #  映射也叫 Mapping，定义了索引中文档的“字段结构与类型”，是搜索行为的基础。搞清楚 Mapping，很多“为什么搜不到/搜不准”的问题就会迎刃而解。
核心概念 #  Mapping 负责什么？ #   字段类型：text / keyword / 数值 / 日期 / 布尔 / 对象 / nested 等 字段是否可搜索、是否可排序/聚合（doc_values、fielddata 等） 是否启用 _source、动态映射策略等  可以认为 Mapping 是 Easysearch 的&quot;模式（schema）&quot;，但比传统数据库更灵活。
精确值 vs 全文 #  在 Easysearch 里，大致可以把字段分成两类：
 精确值：ID、日期、数值、状态码、枚举、用户名、邮箱等  Foo 与 foo 被视为不同 2014 与 2014-09-15 也被视为不同值   全文：自然语言文本，如标题、正文、评论内容等  我们更关心“匹不匹配、相关度高不高”，而不是完全相同    精确值的查询语义更像 SQL 里的：
WHERE name = &#34;John Smith&#34; AND user_id = 2 AND date &gt; &#34;2014-09-15&#34; 全文则是“相关性排序”的世界：搜索的是“像什么”“更接近什么”，而不是绝对等号。这也是为什么同样是字符串字段，有的该按精确值建模，有的该按全文建模。"
---


# 映射基础

映射也叫 Mapping，定义了索引中文档的“字段结构与类型”，是搜索行为的基础。搞清楚 Mapping，很多“为什么搜不到/搜不准”的问题就会迎刃而解。

## 核心概念

### Mapping 负责什么？

- 字段类型：text / keyword / 数值 / 日期 / 布尔 / 对象 / nested 等
- 字段是否可搜索、是否可排序/聚合（doc_values、fielddata 等）
- 是否启用 `_source`、动态映射策略等

可以认为 Mapping 是 Easysearch 的"模式（schema）"，但比传统数据库更灵活。

### 精确值 vs 全文

在 Easysearch 里，大致可以把字段分成两类：

- **精确值**：ID、日期、数值、状态码、枚举、用户名、邮箱等
  - `Foo` 与 `foo` 被视为不同
  - `2014` 与 `2014-09-15` 也被视为不同值
- **全文**：自然语言文本，如标题、正文、评论内容等
  - 我们更关心“匹不匹配、相关度高不高”，而不是完全相同

精确值的查询语义更像 SQL 里的：

```sql
WHERE name    = "John Smith"
  AND user_id = 2
  AND date    > "2014-09-15"
```

全文则是“相关性排序”的世界：搜索的是“像什么”“更接近什么”，而不是绝对等号。这也是为什么同样是字符串字段，有的该按精确值建模，有的该按全文建模。

### text vs keyword：最重要的一对类型

- `text`：
  - 写入时会被分析器分词、归一化
  - 适合全文检索（match/match_phrase 等）
  - 不适合直接排序/聚合（需要专门配置）
- `keyword`：
  - 不分词，整体作为一个值
  - 适合精确过滤、排序、聚合（terms、sort）
  - 不适合做自然语言级别的模糊匹配

人类可读字符串字段，通常需要同时提供 text + keyword 两个视角（详见“Mapping 模式与最佳实践”）。

## 设计与选择

### 字段类型选择：一个简单决策树

可以先按下面几步思考：

1. **这是“描述信息”还是“精确属性”？**
   - 描述信息（标题、正文、评论等） → `text` 为主，考虑是否需要额外 `keyword`
   - 精确属性（ID、状态、枚举、code） → `keyword` / 数值 / 布尔 / 日期
2. **将来要不要排序 / 聚合？**
   - 需要：字段类型必须支持 doc_values（`keyword`、数值、日期、布尔等）
   - 纯全文相关性，不排序不聚合：`text` 即可
3. **是否要支持多种视角（全文 + 精确）？**
   - 需要：用 multi-fields 同时暴露 `field`（text）与 `field.keyword`（keyword）

几个典型场景：

- 日志消息：`message` → `text`，必要时再加 `message.keyword` 方便聚合/精确过滤
- 订单：`order_id` → `keyword`；`status` → `keyword`；`amount` → 数值；`created_at` → `date`
- 商品：`title` → `text` + `title.keyword`；`brand` → `keyword`；`price` → 数值；`tags` → `keyword` 数组

### 动态映射与显式映射

默认情况下，索引可以使用**动态映射**：当新字段第一次出现在文档中时，系统会尝试根据值推断字段类型。

优点：

- 上手快，不需要一开始写完整的 Mapping

缺点：

- 推断结果不一定符合预期（例如数字字符串被当成数值）
- 字段名稍有差异就会产生大量字段，影响管理与性能

建议：

- 对核心业务索引，尽量为关键字段写**显式 Mapping**
- 可以配合动态模板限制或规范动态映射行为

## 字段类型速览

### 常见字段类型概览

- 字符串：
  - `text`：全文检索
  - `keyword`：精确匹配、排序、聚合
- 数值：
  - `integer`、`long`、`float`、`double` 等
- 日期：
  - `date`：支持多种格式，适合时间范围查询与聚合
- 布尔：
  - `boolean`：true/false
- 结构：
  - `object`：嵌套对象（扁平存储，多值字段）
  - `nested`：真正的嵌套文档，支持精确的子对象查询（见“数据建模”章节）

### multi-fields：一份数据，多种索引视角

multi-fields 允许你在 Mapping 中为同一源字段定义多个“索引视图”。最常见的模式是：**全文 + 精确值**：

```json
PUT /products
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",              // 用于全文检索
        "fields": {
          "keyword": {
            "type": "keyword"        // 用于过滤、聚合、排序
          }
        }
      }
    }
  }
}
```

查询时的典型用法：

- `title`：`match` / `multi_match` 等全文检索
- `title.keyword`：`term/terms` + 聚合 + 排序

## 总结与进阶

### 重点回顾

- Mapping 是所有查询/聚合行为的基础，设计字段类型时要先想清楚"将来如何用这些字段"
- 对重要索引，应使用显式 Mapping 管控字段类型与命名
- text/keyword 的区别要牢记，这是绝大多数搜索问题的根源之一

下一步可以继续阅读：

- [映射模式与常见坑]({{< relref "/docs/best-practices/data-modeling/mapping-patterns.md" >}})
- [Query DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})
- [结构化搜索]({{< relref "/docs/features/query-dsl/structured-search.md" >}})

### 参考手册与 API

- [字段类型总览（功能手册）]({{< relref "docs/features/mapping-and-analysis/field-types/_index.md" >}})
- [映射参数（功能手册）]({{< relref "docs/features/mapping-and-analysis/mapping-parameters/_index.md" >}})
- [元字段（功能手册）]({{< relref "docs/features/mapping-and-analysis/metadata-field/_index.md" >}})


