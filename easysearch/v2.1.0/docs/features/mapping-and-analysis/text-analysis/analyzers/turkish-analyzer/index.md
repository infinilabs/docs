---
title: "土耳其语分析器（Turkish）"
date: 0001-01-01
summary: "Turkish 分析器 #  turkish 分析器是为土耳其语文本特别设计的语言分析器，使用土耳其语专用的小写转换。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 apostrophe 分词过滤器：移除撇号及其后字符 turkish_lowercase 分词过滤器：土耳其语专用小写转换（正确处理 İ/I） stop 分词过滤器：过滤土耳其语停用词 snowball(Turkish) 分词过滤器：土耳其语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;turkish&#34;, &#34;text&#34;: &#34;Köpekler parkta koşuyor&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _turkish_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Turkish 分析器

`turkish` 分析器是为土耳其语文本特别设计的语言分析器，使用土耳其语专用的小写转换。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `apostrophe` 分词过滤器：移除撇号及其后字符
- `turkish_lowercase` 分词过滤器：土耳其语专用小写转换（正确处理 İ/I）
- `stop` 分词过滤器：过滤土耳其语停用词
- `snowball`(Turkish) 分词过滤器：土耳其语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "turkish",
  "text": "Köpekler parkta koşuyor"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_turkish_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

