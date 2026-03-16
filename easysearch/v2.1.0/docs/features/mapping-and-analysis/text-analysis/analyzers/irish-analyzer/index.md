---
title: "爱尔兰语分析器（Irish）"
date: 0001-01-01
summary: "Irish 分析器 #  irish 分析器是为爱尔兰语文本特别设计的语言分析器，使用爱尔兰语专用的小写转换和双重停用词过滤。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 stop(hyphenations) 分词过滤器：移除连字符 elision 分词过滤器：移除爱尔兰语省音符号 irish_lowercase 分词过滤器：爱尔兰语专用小写转换 stop 分词过滤器：过滤爱尔兰语停用词 snowball(Irish) 分词过滤器：爱尔兰语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;irish&#34;, &#34;text&#34;: &#34;Rithfidh na madraí sa pháirc&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _irish_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Irish 分析器

`irish` 分析器是为爱尔兰语文本特别设计的语言分析器，使用爱尔兰语专用的小写转换和双重停用词过滤。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `stop`(hyphenations) 分词过滤器：移除连字符
- `elision` 分词过滤器：移除爱尔兰语省音符号
- `irish_lowercase` 分词过滤器：爱尔兰语专用小写转换
- `stop` 分词过滤器：过滤爱尔兰语停用词
- `snowball`(Irish) 分词过滤器：爱尔兰语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "irish",
  "text": "Rithfidh na madraí sa pháirc"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_irish_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

