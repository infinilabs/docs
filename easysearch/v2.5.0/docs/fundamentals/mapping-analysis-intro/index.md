---
title: "Mapping 与文本分析"
date: 0001-01-01
description: "精确值与全文、映射类型、分析器原理与语言分析器选择。"
summary: "Mapping 与文本分析 #  Easysearch 中不同字段类型的索引和搜索行为截然不同。理解映射（Mapping）和分析（Analysis）是用好全文搜索的关键。
精确值 VS 全文 #  Easysearch 中的数据可以概括为两类：
精确值（Exact Values）
 例如日期 2024-09-15、用户 ID user_123、状态码 published Foo 和 foo 是不同的，2024 和 2024-09-15 也是不同的 查询是二进制的：要么匹配，要么不匹配  全文（Full Text）
 例如文章内容、商品描述、邮件正文 查询不是&quot;是否匹配&quot;，而是&quot;匹配程度有多大&quot; 期望搜索 jump 能匹配 jumped、jumps、jumping 期望搜索 UK 能返回包含 United Kingdom 的文档  为了支持全文搜索，Easysearch 在索引之前会先对文本进行分析，生成标准化的词条后写入倒排索引。
分析器：三步处理 #  分析（Analysis） 是将文本转换为适合倒排索引的词条的过程。一个分析器由三个部分组成：
   组件 作用 示例     字符过滤器 在分词前整理字符串 去掉 HTML 标签、&amp; → and   分词器 将字符串拆分为单独的词条 按空格/标点拆分   Token 过滤器 对词条做变换 小写化、词干提取、同义词扩展、停用词删除    内置分析器对比 #  以文本 &quot;Set the shape to semi-transparent by calling set_trans(5)&quot; 为例："
---


# Mapping 与文本分析

Easysearch 中不同字段类型的索引和搜索行为截然不同。理解映射（Mapping）和分析（Analysis）是用好全文搜索的关键。

## 精确值 VS 全文

Easysearch 中的数据可以概括为两类：

**精确值（Exact Values）**
- 例如日期 `2024-09-15`、用户 ID `user_123`、状态码 `published`
- `Foo` 和 `foo` 是不同的，`2024` 和 `2024-09-15` 也是不同的
- 查询是二进制的：要么匹配，要么不匹配

**全文（Full Text）**
- 例如文章内容、商品描述、邮件正文
- 查询不是"是否匹配"，而是"匹配程度有多大"
- 期望搜索 `jump` 能匹配 `jumped`、`jumps`、`jumping`
- 期望搜索 `UK` 能返回包含 `United Kingdom` 的文档

为了支持全文搜索，Easysearch 在索引之前会先对文本进行**分析**，生成标准化的词条后写入倒排索引。

## 分析器：三步处理

**分析（Analysis）** 是将文本转换为适合倒排索引的词条的过程。一个分析器由三个部分组成：

| 组件 | 作用 | 示例 |
|------|------|------|
| **字符过滤器** | 在分词前整理字符串 | 去掉 HTML 标签、`&` → `and` |
| **分词器** | 将字符串拆分为单独的词条 | 按空格/标点拆分 |
| **Token 过滤器** | 对词条做变换 | 小写化、词干提取、同义词扩展、停用词删除 |

### 内置分析器对比

以文本 `"Set the shape to semi-transparent by calling set_trans(5)"` 为例：

| 分析器 | 产生的词条 |
|--------|-----------|
| **标准分析器**（默认） | `set, the, shape, to, semi, transparent, by, calling, set_trans, 5` |
| **简单分析器** | `set, the, shape, to, semi, transparent, by, calling, set, trans` |
| **空格分析器** | `Set, the, shape, to, semi-transparent, by, calling, set_trans(5)` |
| **英语分析器** | `set, shape, semi, transpar, call, set_tran, 5` |

### 测试分析器

使用 `_analyze` API 查看文本被分析后的结果：

```json
GET /_analyze
{
  "analyzer": "standard",
  "text": "Quick brown foxes"
}
```

返回：

```json
{
  "tokens": [
    { "token": "quick",  "position": 0 },
    { "token": "brown",  "position": 1 },
    { "token": "foxes",  "position": 2 }
  ]
}
```

也可以针对已有索引的某个字段测试：

```json
GET /my_index/_analyze
{
  "field": "content",
  "text": "Quick brown foxes"
}
```

## 语言分析器选择

Easysearch 为很多流行语言提供了开箱即用的语言分析器。每个语言分析器都针对该语言的特点进行了优化：

- **英语分析器**：移除了所有格 `'s`（`John's` → `john`）、词干提取
- **法语分析器**：移除了元音省略（如 `l'` 和 `qu'`）和变音符号
- **德语分析器**：规范化了切词，将 `ä` 和 `ae` 替换为 `a`，或将 `ß` 替换为 `ss`
- **中文分析器**（IK、Pinyin）：分词与拼音支持

## 映射：字段的类型定义

映射（Mapping）定义了索引中每个字段的数据类型和处理方式。

### 核心字段类型

| 类型 | 说明 | 用途 |
|------|------|------|
| `text` | 全文字段，会被分析后建立倒排索引 | 标题、正文、描述 |
| `keyword` | 精确值字段，不分析，用于过滤、排序、聚合 | 状态、分类、ID、邮箱 |
| `long`, `integer`, `short`, `byte` | 整数类型 | 计数、编号 |
| `double`, `float`, `half_float` | 浮点数类型 | 价格、评分 |
| `boolean` | 布尔类型 | 是否标志 |
| `date` | 日期类型 | 时间戳 |
| `object` | JSON 对象 | 嵌套数据 |

### 动态映射

当你索引一个包含新字段的文档时，Easysearch 会自动猜测字段类型：

| JSON 值 | 推断的类型 |
|---------|-----------|
| `true` / `false` | `boolean` |
| `123` | `long` |
| `123.45` | `double` |
| `"2024-09-15"` | `date` |
| `"hello world"` | `text` + `keyword` 子字段 |

> ⚠️ 如果通过引号索引一个数字（如 `"123"`），它会被映射为 `text`。类型一旦映射后不能修改，只能增加新字段。

### 创建与修改映射

创建索引时指定映射：

```json
PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      },
      "status": {
        "type": "keyword"
      },
      "created_at": {
        "type": "date"
      },
      "views": {
        "type": "integer"
      }
    }
  }
}
```

增加新字段（不能修改已有字段）：

```json
PUT /my_index/_mapping
{
  "properties": {
    "tag": {
      "type": "keyword"
    }
  }
}
```

### 查看映射

```json
GET /my_index/_mapping
```

## 复杂数据类型

### 数组

任何字段都可以包含多个值（数组），不需要特殊映射：

```json
{ "tag": ["search", "nosql"] }
```

> ⚠️ 数组中所有值必须是相同数据类型。数组被索引为多值字段——可搜索，但无序。

### 内部对象

内部对象会被 Lucene"扁平化"索引。例如：

```json
{
  "user": {
    "name": { "first": "John", "last": "Smith" },
    "age": 26
  }
}
```

实际被索引为：

```json
{
  "user.name.first": ["john"],
  "user.name.last":  ["smith"],
  "user.age":        [26]
}
```

内部对象可以通过点号路径引用（如 `user.name.first`）。

> ⚠️ 如果内部对象是数组，扁平化会导致对象间的关联性丢失。如果需要保持对象间关联，请使用 `nested` 类型。

## 关键原则

- **索引和查询用相同的分析器**：确保词条格式一致才能匹配
- **精确值字段不需要分析**：用 `keyword` 类型
- **映射一旦创建不能修改**：只能增加新字段
- **用 `_analyze` API 验证分析效果**：调试分析器的最佳工具

## 小结

| 概念 | 要点 |
|------|------|
| 精确值 | 不分析，用 `keyword`/数值/日期，二进制匹配 |
| 全文 | 通过分析器生成词条，倒排索引，相关性匹配 |
| 分析器 | 字符过滤 → 分词 → Token 过滤，三步流水线 |
| 映射 | 定义字段类型和分析方式，创建后不可改只可加 |

## 下一步

- [Mapping 基础深入](../features/mapping-and-analysis/mapping-basics.md)：text/keyword 策略、动态映射陷阱
- [Mapping 模式与最佳实践](../best-practices/data-modeling/mapping-patterns.md)：多字段设计、嵌套数据
- [文本分析主题](../features/mapping-and-analysis/text-analysis/):词汇识别、词干提取、同义词等
- [文本分析 API](../features/mapping-and-analysis/text-analysis/)：分析器、分词器、过滤器参考
- [Lucene 底层原理](./lucene-internals.md)：倒排索引与段结构

### 最佳实践

- [数据建模]({{< relref "/docs/best-practices/data-modeling/" >}})：文档设计、Nested、反范式权衡
- [索引设计]({{< relref "/docs/best-practices/index-design.md" >}})：分片规划、Mapping 模板优化

