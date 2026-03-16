---
title: "加泰罗尼亚语分析器（Catalan）"
date: 0001-01-01
summary: "Catalan 分析器 #  catalan 分析器是为加泰罗尼亚语文本特别设计的语言分析器，包含省音处理。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 elision 分词过滤器：移除加泰罗尼亚语省音符号 lowercase 分词过滤器：转换为小写 stop 分词过滤器：过滤加泰罗尼亚语停用词 snowball(Catalan) 分词过滤器：加泰罗尼亚语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;catalan&#34;, &#34;text&#34;: &#34;Els gossos corren al parc&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _catalan_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Catalan 分析器

`catalan` 分析器是为加泰罗尼亚语文本特别设计的语言分析器，包含省音处理。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `elision` 分词过滤器：移除加泰罗尼亚语省音符号
- `lowercase` 分词过滤器：转换为小写
- `stop` 分词过滤器：过滤加泰罗尼亚语停用词
- `snowball`(Catalan) 分词过滤器：加泰罗尼亚语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "catalan",
  "text": "Els gossos corren al parc"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_catalan_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

