---
title: "印度尼西亚语分析器（Indonesian）"
date: 0001-01-01
summary: "Indonesian 分析器 #  indonesian 分析器是为印度尼西亚语文本特别设计的语言分析器。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 lowercase 分词过滤器：转换为小写 stop 分词过滤器：过滤印度尼西亚语停用词 indonesian_stem 分词过滤器：印度尼西亚语词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;indonesian&#34;, &#34;text&#34;: &#34;Anjing-anjing berlari di taman&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _indonesian_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Indonesian 分析器

`indonesian` 分析器是为印度尼西亚语文本特别设计的语言分析器。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `lowercase` 分词过滤器：转换为小写
- `stop` 分词过滤器：过滤印度尼西亚语停用词
- `indonesian_stem` 分词过滤器：印度尼西亚语词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "indonesian",
  "text": "Anjing-anjing berlari di taman"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_indonesian_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

