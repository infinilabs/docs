---
title: "分析器（Analyzers）"
date: 0001-01-01
description: "Easysearch 内置分析器参考：Standard、IK、Pinyin、Language 等。"
summary: "分析器（Analyzers） #  分析器是文本分析的核心组件，由三部分组成：
 字符过滤器（0~多个）：预处理原始文本 分词器（必须 1 个）：将文本拆分为词项 词元过滤器（0~多个）：对词项进行后处理  Easysearch 提供多种内置分析器，也支持创建自定义分析器。
相关指南 #    文本分析基础 - 分析器原理  词汇识别 - 分词器选择  词元归一化 - 归一化处理   内置分析器总览 #  通用分析器 #     分析器 说明 适用场景      Standard（标准） 默认分析器，按单词边界分词，转小写 通用英文/欧洲语言    Simple（简单） 按非字母字符分词，转小写 简单文本    Whitespace（空白） 仅按空白字符分词，保留大小写和标点 需要保留原始形式    Stop（停用词） Simple + 移除停用词 需要过滤常用词    Keyword（关键字） 不分词，整体作为一个词项 精确匹配场景    Pattern（正则） 按正则表达式分词 自定义分割规则    Fingerprint（指纹） 去重、排序、连接为单个词项 去重检测    语言分析器 #   完整说明及通用参数请参阅 语言分析器总览。"
---


# 分析器（Analyzers）

分析器是文本分析的核心组件，由三部分组成：

1. **字符过滤器**（0~多个）：预处理原始文本
2. **分词器**（必须 1 个）：将文本拆分为词项
3. **词元过滤器**（0~多个）：对词项进行后处理

Easysearch 提供多种内置分析器，也支持创建自定义分析器。

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}}) - 分析器原理
- [词汇识别]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words" >}}) - 分词器选择
- [词元归一化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization" >}}) - 归一化处理

---

## 内置分析器总览

### 通用分析器

| 分析器 | 说明 | 适用场景 |
|-------|------|---------|
| [Standard（标准）]({{< relref "./standard-analyzer" >}}) | 默认分析器，按单词边界分词，转小写 | 通用英文/欧洲语言 |
| [Simple（简单）]({{< relref "./simple-analyzer" >}}) | 按非字母字符分词，转小写 | 简单文本 |
| [Whitespace（空白）]({{< relref "./whitespace-analyzer" >}}) | 仅按空白字符分词，保留大小写和标点 | 需要保留原始形式 |
| [Stop（停用词）]({{< relref "./stop-analyzer" >}}) | Simple + 移除停用词 | 需要过滤常用词 |
| [Keyword（关键字）]({{< relref "./keyword-analyzer" >}}) | 不分词，整体作为一个词项 | 精确匹配场景 |
| [Pattern（正则）]({{< relref "./pattern-analyzer" >}}) | 按正则表达式分词 | 自定义分割规则 |
| [Fingerprint（指纹）]({{< relref "./fingerprint-analyzer" >}}) | 去重、排序、连接为单个词项 | 去重检测 |

### 语言分析器

> 完整说明及通用参数请参阅 [语言分析器总览]({{< relref "./language-analyzer" >}})。

| 分析器 | 语言 | 说明 |
|-------|------|------|
| [`arabic`]({{< relref "./arabic-analyzer" >}}) | 阿拉伯语 | 阿拉伯语归一化 + 词干提取 |
| [`armenian`]({{< relref "./armenian-analyzer" >}}) | 亚美尼亚语 | Snowball 词干提取 |
| [`basque`]({{< relref "./basque-analyzer" >}}) | 巴斯克语 | Snowball 词干提取 |
| [`bengali`]({{< relref "./bengali-analyzer" >}}) | 孟加拉语 | 印度语系归一化 + 词干提取 |
| [`brazilian`]({{< relref "./brazilian-analyzer" >}}) | 巴西葡萄牙语 | 巴西葡萄牙语词干提取 |
| [`bulgarian`]({{< relref "./bulgarian-analyzer" >}}) | 保加利亚语 | 保加利亚语词干提取 |
| [`catalan`]({{< relref "./catalan-analyzer" >}}) | 加泰罗尼亚语 | 省音处理 + Snowball 词干提取 |
| [`czech`]({{< relref "./czech-analyzer" >}}) | 捷克语 | 捷克语词干提取 |
| [`danish`]({{< relref "./danish-analyzer" >}}) | 丹麦语 | Snowball 词干提取 |
| [`dutch`]({{< relref "./dutch-analyzer" >}}) | 荷兰语 | 词干覆盖字典 + Snowball |
| [`english`]({{< relref "./english-analyzer" >}}) | 英语 | 所有格处理 + Porter 词干提取 |
| [`estonian`]({{< relref "./estonian-analyzer" >}}) | 爱沙尼亚语 | Snowball 词干提取 |
| [`finnish`]({{< relref "./finnish-analyzer" >}}) | 芬兰语 | Snowball 词干提取 |
| [`french`]({{< relref "./french-analyzer" >}}) | 法语 | 省音处理 + 轻量词干提取 |
| [`galician`]({{< relref "./galician-analyzer" >}}) | 加利西亚语 | 加利西亚语词干提取 |
| [`german`]({{< relref "./german-analyzer" >}}) | 德语 | 字符归一化 + 轻量词干提取 |
| [`greek`]({{< relref "./greek-analyzer" >}}) | 希腊语 | 希腊语专用小写 + 词干提取 |
| [`hindi`]({{< relref "./hindi-analyzer" >}}) | 印地语 | 印度语系归一化 + 词干提取 |
| [`hungarian`]({{< relref "./hungarian-analyzer" >}}) | 匈牙利语 | Snowball 词干提取 |
| [`indonesian`]({{< relref "./indonesian-analyzer" >}}) | 印度尼西亚语 | 印度尼西亚语词干提取 |
| [`irish`]({{< relref "./irish-analyzer" >}}) | 爱尔兰语 | 专用小写 + 双重停用词过滤 |
| [`italian`]({{< relref "./italian-analyzer" >}}) | 意大利语 | 省音处理 + 轻量词干提取 |
| [`latvian`]({{< relref "./latvian-analyzer" >}}) | 拉脱维亚语 | 拉脱维亚语词干提取 |
| [`lithuanian`]({{< relref "./lithuanian-analyzer" >}}) | 立陶宛语 | Snowball 词干提取 |
| [`norwegian`]({{< relref "./norwegian-analyzer" >}}) | 挪威语 | Snowball 词干提取 |
| [`persian`]({{< relref "./persian-analyzer" >}}) | 波斯语 | 字符过滤 + 阿拉伯语/波斯语归一化 |
| [`polish`]({{< relref "./polish-analyzer" >}}) | 波兰语 | 波兰语词干提取 |
| [`portuguese`]({{< relref "./portuguese-analyzer" >}}) | 葡萄牙语 | 轻量词干提取 |
| [`romanian`]({{< relref "./romanian-analyzer" >}}) | 罗马尼亚语 | 字符归一化 + Snowball 词干提取 |
| [`russian`]({{< relref "./russian-analyzer" >}}) | 俄语 | Snowball 词干提取 |
| [`sorani`]({{< relref "./sorani-analyzer" >}}) | 索拉尼语 | 索拉尼语归一化 + 词干提取 |
| [`spanish`]({{< relref "./spanish-analyzer" >}}) | 西班牙语 | 轻量词干提取 |
| [`swedish`]({{< relref "./swedish-analyzer" >}}) | 瑞典语 | Snowball 词干提取 |
| [`thai`]({{< relref "./thai-analyzer" >}}) | 泰语 | Java BreakIterator 泰语分词 |
| [`turkish`]({{< relref "./turkish-analyzer" >}}) | 土耳其语 | 专用小写（İ/I）+ Snowball |

### 形态分析器

| 分析器 | 说明 |
|-------|------|
| [English Morphology]({{< relref "./english-morphology-analyzer" >}}) | 英语形态分析 |
| [Russian Morphology]({{< relref "./russian-morphology-analyzer" >}}) | 俄语形态分析 |

### 中文分析器

| 分析器 | 说明 | 适用场景 |
|-------|------|---------|
| [IK 中文分析器]({{< relref "./ik-analyzer" >}}) | 智能中文分词（ik_smart / ik_max_word） | 中文全文搜索 |
| [Pinyin（拼音）]({{< relref "./pinyin-analyzer" >}}) | 中文转拼音 | 拼音搜索、输入法联想 |
| [STConvert（简繁转换）]({{< relref "./stconvert-analyzer" >}}) | 简体繁体互转 | 跨简繁搜索 |

## 选择分析器

| 场景 | 推荐分析器 |
|------|-----------|
| 英文全文搜索 | `standard` 或 `english` |
| 中文全文搜索 | `ik_smart`（平衡）或 `ik_max_word`（细粒度） |
| 精确匹配 | `keyword` |
| 保留标点/格式 | `whitespace` |
| 拼音搜索 | `pinyin` |
| 多语言混合 | 自定义分析器组合多种过滤器 |

## 创建自定义分析器

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": ["lowercase", "porter_stem"]
        }
      }
    }
  }
}
```

## 测试分析器

```json
// 测试内置分析器
GET /_analyze
{
  "analyzer": "standard",
  "text": "The quick brown fox"
}

// 测试索引的分析器
GET /my_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "The quick brown fox"
}
```
