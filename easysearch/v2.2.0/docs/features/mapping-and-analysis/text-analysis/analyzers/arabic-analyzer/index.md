---
title: "阿拉伯语分析器（Arabic）"
date: 0001-01-01
summary: "Arabic 分析器 #  arabic 分析器是为阿拉伯语文本特别设计的语言分析器，包含了阿拉伯语特有的归一化和词干提取规则。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 lowercase 分词过滤器：转换为小写 decimal_digit 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字 stop 分词过滤器：过滤阿拉伯语停用词 arabic_normalization 分词过滤器：阿拉伯语字符归一化 arabic_stem 分词过滤器：阿拉伯语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;arabic&#34;, &#34;text&#34;: &#34;الكلاب تركض في الحديقة&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _arabic_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Arabic 分析器

`arabic` 分析器是为阿拉伯语文本特别设计的语言分析器，包含了阿拉伯语特有的归一化和词干提取规则。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `lowercase` 分词过滤器：转换为小写
- `decimal_digit` 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字
- `stop` 分词过滤器：过滤阿拉伯语停用词
- `arabic_normalization` 分词过滤器：阿拉伯语字符归一化
- `arabic_stem` 分词过滤器：阿拉伯语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "arabic",
  "text": "الكلاب تركض في الحديقة"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_arabic_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

