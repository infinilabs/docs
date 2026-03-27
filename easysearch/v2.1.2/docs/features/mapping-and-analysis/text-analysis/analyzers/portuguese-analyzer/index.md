---
title: "葡萄牙语分析器（Portuguese）"
date: 0001-01-01
summary: "Portuguese 分析器 #  portuguese 分析器是为葡萄牙语文本特别设计的语言分析器。
分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 standard 分词器：标准的文本分割 lowercase 分词过滤器：转换为小写 stop 分词过滤器：过滤葡萄牙语停用词 portuguese_light_stem 分词过滤器：葡萄牙语轻量词干提取  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;portuguese&#34;, &#34;text&#34;: &#34;Os cães correm no parque&#34; } 自定义配置 #  可通过以下参数自定义该分析器：
   参数 说明     stopwords 自定义停用词列表，默认 _portuguese_   stem_exclusion 不进行词干提取的词语列表    相关指南 #    语言分析器  文本分析基础  "
---


# Portuguese 分析器

`portuguese` 分析器是为葡萄牙语文本特别设计的语言分析器。

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：标准的文本分割
- `lowercase` 分词过滤器：转换为小写
- `stop` 分词过滤器：过滤葡萄牙语停用词
- `portuguese_light_stem` 分词过滤器：葡萄牙语轻量词干提取

## 示例

```json
POST _analyze
{
  "analyzer": "portuguese",
  "text": "Os cães correm no parque"
}
```

## 自定义配置

可通过以下参数自定义该分析器：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认 `_portuguese_` |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [语言分析器]({{< relref "./language-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

