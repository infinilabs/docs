---
title: "意大利语分析器（Italian）"
date: 0001-01-01
summary: "Italian 分析器 #  italian 分析器是为意大利语文本特别设计的语言分析器，包含省音处理。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 elision 分词过滤器：移除意大利语省音符号 lowercase 分词过滤器：转换为小写 stop 分词过滤器：过滤意大利语停用词 italian_light_stem 分词过滤器：意大利语轻量词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;italian&#34;, &#34;text&#34;: &#34;I cani corrono nel parco&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _italian_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Italian 分析器

`italian` 分析器是为意大利语文本特别设计的语言分析器，包含省音处理。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `elision` 分词过滤器：移除意大利语省音符号
- `lowercase` 分词过滤器：转换为小写
- `stop` 分词过滤器：过滤意大利语停用词
- `italian_light_stem` 分词过滤器：意大利语轻量词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "italian",
  "text": "I cani corrono nel parco"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_italian_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

