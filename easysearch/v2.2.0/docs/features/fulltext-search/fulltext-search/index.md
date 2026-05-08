---
title: "全文检索"
date: 0001-01-01
description: "match/match_phrase/multi_match 等全文检索查询的用法与注意事项。"
summary: "全文检索 #  全文检索的核心特点是：对 text 字段做分词与评分，用“相关性”来排序结果。本页只关注最常用的几类查询及常见坑。
前提：字段必须是可分析（text）类型 #  全文查询（match/match_phrase/multi_match 等）应该作用在 text 字段 上：
 写入时会通过 analyzer 做分词、归一化（大小写、同义词等） 查询时会用同一个 analyzer 处理查询词，再去匹配倒排索引  如果字段是 keyword/数值/日期，更适合使用 Term 级别查询。
match：最常用的全文查询 #  match 会：
 对查询字符串分词 按字段的 analyzer 处理 把多个词项组合成一个全文查询，并参与 _score 计算  示例：
{ &#34;query&#34;: { &#34;match&#34;: { &#34;title&#34;: &#34;分布式 搜索 引擎&#34; } } } 常见参数：
 operator：or（默认）或 and minimum_should_match：要求最少命中多少词  一个直觉对比：
 operator: &quot;and&quot;：所有词都必须出现，召回会明显变少，但结果通常更“干净” minimum_should_match: &quot;75%&quot;：允许部分词缺失，在“可搜到”和“不要太多噪声”之间找折中  示例（用户搜索“分布式 搜索 引擎 调优”）："
---


# 全文检索

全文检索的核心特点是：**对 text 字段做分词与评分**，用“相关性”来排序结果。本页只关注最常用的几类查询及常见坑。

## 前提：字段必须是可分析（text）类型

全文查询（`match`/`match_phrase`/`multi_match` 等）应该作用在 **text 字段** 上：

- 写入时会通过 analyzer 做分词、归一化（大小写、同义词等）
- 查询时会用同一个 analyzer 处理查询词，再去匹配倒排索引

如果字段是 keyword/数值/日期，更适合使用[Term 级别查询]({{< relref "/docs/features/query-dsl/term-based-query/" >}})。

## match：最常用的全文查询

`match` 会：

- 对查询字符串分词
- 按字段的 analyzer 处理
- 把多个词项组合成一个全文查询，并参与 `_score` 计算

示例：

```json
{
  "query": {
    "match": {
      "title": "分布式 搜索 引擎"
    }
  }
}
```

常见参数：

- `operator`：`or`（默认）或 `and`
- `minimum_should_match`：要求最少命中多少词

一个直觉对比：

- `operator: "and"`：所有词都必须出现，召回会明显变少，但结果通常更“干净”
- `minimum_should_match: "75%"`：允许部分词缺失，在“可搜到”和“不要太多噪声”之间找折中

示例（用户搜索“分布式 搜索 引擎 调优”）：

```json
{
  "match": {
    "title": {
      "query": "分布式 搜索 引擎 调优",
      "minimum_should_match": "75%"
    }
  }
}
```

这样“差一个词”的结果还能被召回，但完全不相关的长条内容会被剪掉。

## match_phrase：短语匹配（词序 + 距离）

`match_phrase` 适合“词与词之间顺序和距离都很重要”的场景，例如：

- 人名、书名、精确短语
- “搜索 引擎” vs “引擎 搜索” 要区分

示例：

```json
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "分布式 搜索 引擎",
        "slop": 1
      }
    }
  }
}
```

- `slop` 表示允许的词间“位移”（更大的 slop 意味着更宽松，但可能匹配更多噪声）。

## multi_match：在多个字段上搜同一串词

适用于“用户输入一句话，你想同时在多个字段上查”的场景，例如 title + content + tags：

```json
{
  "query": {
    "multi_match": {
      "query": "日志 搜索 性能",
      "fields": ["title^3", "content", "tags^2"],
      "type": "best_fields"
    }
  }
}
```

说明：

- `fields`：字段列表，可以用 `^` 做字段级 boost（`title^3` 表示标题权重更高）
- `type` 常用：
  - `best_fields`：看哪一个字段匹配得最好（默认）
  - `most_fields`：多个字段共同贡献相关性

## 简单选择策略

- 只在单字段上搜：优先用 `match`
- 在一句话级文本上搜“短语”：用 `match_phrase`，必要时调整 `slop`
- 在多字段上搜同一串词：用 `multi_match`，配合字段 boost

如果你已经在用“多字段搜索”套路（标题 + 正文 + 标签），可以把 `multi_match` 理解成“语法糖版的 bool + 多个 match”，更完整的多字段策略见[多字段搜索](./multi-field-search.md)一节。

## 常见坑与注意点

- **对 keyword 字段用 match**：  
  由于 keyword 不分词，match 行为会退化为近似 term，用法容易混乱。  
  → 建议：为需要全文检索的字段提供 text 版本（multi-fields），keyword 只做结构化过滤/聚合。

- **忽略分析器的影响**：  
  查询结果不符合预期时，先用 [Analyze API]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}}) 看一下文本是如何被分词与归一化的。

- **滥用 `operator: and`**：  
  在很多用户输入场景，使用 `and` 容易导致“搜不到任何东西”；更推荐用 `minimum_should_match` 细调。

## 与结构化过滤结合

在实际业务里，全文检索几乎总是和结构化过滤一起用：

```json
{
  "query": {
    "bool": {
      "must": [
        { "multi_match": {
            "query": "搜索 性能",
            "fields": ["title^2", "content"]
        } }
      ],
      "filter": [
        { "term": { "status": "active" } },
        { "range": { "created_at": { "gte": "now-30d/d" } } }
      ]
    }
  }
}
```

全文部分负责"排序谁更相关"，结构化部分负责"限定边界、减少噪声"。

## 小结

- 全文检索基于倒排索引，通过分析器对文本进行分词和归一化
- `match` 查询是最常用的全文检索查询，支持多词匹配和相关性评分
- `multi_match` 可以在多个字段上执行全文检索，支持字段权重提升
- 短语查询和模糊查询可以处理更复杂的匹配需求
- 全文检索通常与结构化过滤结合使用

下一步可以继续阅读：

- [分页与排序](./pagination-and-sorting.md)
- [高亮](./highlighting.md)
- [相关性](./relevance/_index.md)

## 参考手册（API 与参数）

- [全文查询（功能手册）]({{< relref "/docs/features/fulltext-search/full-text/_index.md" >}})
- [搜索操作概览（功能手册）]({{< relref "docs/features/fulltext-search/_index.md" >}})




