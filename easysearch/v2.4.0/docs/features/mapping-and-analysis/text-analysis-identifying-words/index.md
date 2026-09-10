---
title: "词汇识别"
date: 0001-01-01
description: "如何将文本拆分为词项：标准分析器、分词器与不同语言的处理方式。"
summary: "词汇识别 #  词汇识别是文本分析的第一步：将文本拆分为可搜索的词项。不同语言的词汇识别方式差异很大，需要选择合适的分析器和分词器。
不同语言的挑战 #  英语 #  英语单词相对容易识别：单词之间通常以空格或标点分隔。但也有一些边界情况：
 you're 是一个单词还是两个？ o'clock、cooperate、half-baked、eyewitness 等复合词如何处理？  德语和荷兰语 #  这些语言会将独立的单词合并成长复合词，例如：
 Weißkopfseeadler（white-headed sea eagle）  为了在查询 Adler（eagle）时也能匹配到 Weißkopfseeadler，需要将复合词拆分成词组。
亚洲语言 #  亚洲语言更复杂：
 很多语言在单词、句子甚至段落之间没有空格 有些词可以用一个字表达，但同样的字在另一个字旁边时就是不同意思的长词的一部分  标准分析器 #  任何全文检索的 text 字段默认使用 standard 分析器。标准分析器包括：
 分词器：standard 分词器 词元过滤器：lowercase（小写转换）和 stop（停用词，默认关闭）  标准分析器可以这样重新实现：
{ &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;lowercase&#34;, &#34;stop&#34;] } 标准分词器 #  standard 分词器基于 Unicode 文本分割算法，能够：
 识别单词边界（空格、标点） 处理大多数欧洲语言 处理亚洲语言（虽然可能不够精确）  示例："
---


# 词汇识别

词汇识别是文本分析的第一步：将文本拆分为可搜索的词项。不同语言的词汇识别方式差异很大，需要选择合适的分析器和分词器。

## 不同语言的挑战

### 英语

英语单词相对容易识别：单词之间通常以空格或标点分隔。但也有一些边界情况：

- `you're` 是一个单词还是两个？
- `o'clock`、`cooperate`、`half-baked`、`eyewitness` 等复合词如何处理？

### 德语和荷兰语

这些语言会将独立的单词合并成长复合词，例如：

- `Weißkopfseeadler`（white-headed sea eagle）

为了在查询 `Adler`（eagle）时也能匹配到 `Weißkopfseeadler`，需要将复合词拆分成词组。

### 亚洲语言

亚洲语言更复杂：

- 很多语言在单词、句子甚至段落之间没有空格
- 有些词可以用一个字表达，但同样的字在另一个字旁边时就是不同意思的长词的一部分

## 标准分析器

任何全文检索的 `text` 字段默认使用 `standard` 分析器。标准分析器包括：

- **分词器**：`standard` 分词器
- **词元过滤器**：`lowercase`（小写转换）和 `stop`（停用词，默认关闭）

标准分析器可以这样重新实现：

```json
{
  "type": "custom",
  "tokenizer": "standard",
  "filter": ["lowercase", "stop"]
}
```

### 标准分词器

`standard` 分词器基于 Unicode 文本分割算法，能够：

- 识别单词边界（空格、标点）
- 处理大多数欧洲语言
- 处理亚洲语言（虽然可能不够精确）

示例：

```json
GET /_analyze
{
  "tokenizer": "standard",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

输出：

```
[ The, 2, QUICK, Brown, Foxes, jumped, over, the, lazy, dog's, bone ]
```

可以看到：
- `Brown-Foxes` 被拆分为 `Brown` 和 `Foxes`
- `dog's` 被保留为一个词项（不会拆分）

## 语言特定的分析器

对于特定语言，Easysearch 提供了专门的分析器：

- **中文**：需要中文分词器（如 IK 分词器）
- **日语**：需要日语分词器（如 Kuromoji）
- **韩语**：需要韩语分词器
- **德语**：可以使用德语分析器处理复合词

## ICU 分词器

对于需要更精确 Unicode 支持的语言，可以使用 ICU 分词器：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_icu_analyzer": {
          "tokenizer": "icu_tokenizer"
        }
      }
    }
  }
}
```

ICU 分词器能够：

- 更好地处理 Unicode 文本
- 支持更多语言的文本分割规则
- 处理变音符号和特殊字符

## 选择合适的分词器

选择分词器时需要考虑：

1. **语言类型**：
   - 欧洲语言：`standard` 分词器通常足够
   - 亚洲语言：需要语言特定的分词器
   - 混合语言：可能需要自定义分析器

2. **文本特点**：
   - 是否包含复合词
   - 是否需要保留特殊字符
   - 是否需要处理变音符号

3. **查询需求**：
   - 是否需要精确匹配
   - 是否需要模糊匹配
   - 是否需要处理同义词

## 测试分词器

使用 `_analyze` API 测试分词器的效果：

```json
GET /_analyze
{
  "tokenizer": "standard",
  "text": "The quick brown fox"
}
```

或者测试整个分析器：

```json
GET /my_index/_analyze
{
  "field": "title",
  "text": "The quick brown fox"
}
```

## 小结

- 词汇识别是将文本拆分为词项的第一步
- 不同语言的词汇识别方式差异很大
- `standard` 分词器适合大多数欧洲语言
- 亚洲语言需要语言特定的分词器
- 可以使用 `_analyze` API 测试分词器的效果

下一步可以继续阅读：

- [词元归一化](./text-analysis-normalization.md)
- [词干提取](./text-analysis-stemming.md)
- [文本分析](./text-analysis/)

## 参考手册（API 与参数）

- [分词器（功能手册）]({{< relref "docs/features/mapping-and-analysis/text-analysis/tokenizers/_index.md" >}})

