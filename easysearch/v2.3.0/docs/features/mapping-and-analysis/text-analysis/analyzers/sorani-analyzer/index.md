---
title: "索拉尼语分析器（Sorani）"
date: 0001-01-01
summary: "Sorani 分析器 #  sorani 分析器是为索拉尼库尔德语文本特别设计的语言分析器。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 sorani_normalization 分词过滤器：索拉尼语字符归一化 lowercase 分词过滤器：转换为小写 decimal_digit 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字 stop 分词过滤器：过滤索拉尼语停用词 sorani_stem 分词过滤器：索拉尼语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;sorani&#34;, &#34;text&#34;: &#34;سەگەکان لە پارکەکەدا ڕادەکەن&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _sorani_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Sorani 分析器

`sorani` 分析器是为索拉尼库尔德语文本特别设计的语言分析器。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `sorani_normalization` 分词过滤器：索拉尼语字符归一化
- `lowercase` 分词过滤器：转换为小写
- `decimal_digit` 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字
- `stop` 分词过滤器：过滤索拉尼语停用词
- `sorani_stem` 分词过滤器：索拉尼语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "sorani",
  "text": "سەگەکان لە پارکەکەدا ڕادەکەن"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_sorani_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

