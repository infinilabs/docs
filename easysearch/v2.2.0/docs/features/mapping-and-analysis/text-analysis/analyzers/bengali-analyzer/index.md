---
title: "孟加拉语分析器（Bengali）"
date: 0001-01-01
summary: "Bengali 分析器 #  bengali 分析器是为孟加拉语文本特别设计的语言分析器，包含印度语系归一化处理。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 lowercase 分词过滤器：转换为小写 decimal_digit 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字 indic_normalization 分词过滤器：印度语系字符归一化 bengali_normalization 分词过滤器：孟加拉语字符归一化 stop 分词过滤器：过滤孟加拉语停用词 bengali_stem 分词过滤器：孟加拉语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;bengali&#34;, &#34;text&#34;: &#34;বাংলা ভাষার বিশ্লেষণ&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _bengali_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Bengali 分析器

`bengali` 分析器是为孟加拉语文本特别设计的语言分析器，包含印度语系归一化处理。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `lowercase` 分词过滤器：转换为小写
- `decimal_digit` 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字
- `indic_normalization` 分词过滤器：印度语系字符归一化
- `bengali_normalization` 分词过滤器：孟加拉语字符归一化
- `stop` 分词过滤器：过滤孟加拉语停用词
- `bengali_stem` 分词过滤器：孟加拉语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "bengali",
  "text": "বাংলা ভাষার বিশ্লেষণ"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_bengali_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

