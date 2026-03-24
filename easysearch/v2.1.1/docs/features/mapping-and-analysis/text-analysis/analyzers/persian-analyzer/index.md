---
title: "波斯语分析器（Persian）"
date: 0001-01-01
summary: "Persian 分析器 #  persian 分析器是为波斯语文本特别设计的语言分析器，包含阿拉伯语归一化和波斯语字符过滤。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 persian_char_filter 字符过滤器：将零宽非连接符替换为空格 standard 分词器：标准的文本分割 lowercase 分词过滤器：转换为小写 decimal_digit 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字 arabic_normalization 分词过滤器：阿拉伯语字符归一化 persian_normalization 分词过滤器：波斯语字符归一化 stop 分词过滤器：过滤波斯语停用词  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;persian&#34;, &#34;text&#34;: &#34;سگ‌ها در پارک می‌دوند&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _persian_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Persian 分析器

`persian` 分析器是为波斯语文本特别设计的语言分析器，包含阿拉伯语归一化和波斯语字符过滤。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `persian_char_filter` 字符过滤器：将零宽非连接符替换为空格
- `standard` 分词器：标准的文本分割
- `lowercase` 分词过滤器：转换为小写
- `decimal_digit` 分词过滤器：将各种 Unicode 数字转换为 ASCII 数字
- `arabic_normalization` 分词过滤器：阿拉伯语字符归一化
- `persian_normalization` 分词过滤器：波斯语字符归一化
- `stop` 分词过滤器：过滤波斯语停用词

## 示例

```json
POST _analyze
{
  "analyzer": "persian",
  "text": "سگ‌ها در پارک می‌دوند"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_persian_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

