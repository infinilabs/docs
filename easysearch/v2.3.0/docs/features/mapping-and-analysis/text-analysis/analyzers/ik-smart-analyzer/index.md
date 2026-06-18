---
title: "IK 智能分析器（IK Smart）"
date: 0001-01-01
summary: "IK Smart 分析器 #  ik_smart 分析器是为中文智能分词的分析器，使用 IK 分词器的智能模式，会尽量将文本切分为最少的词语。
需要安装 analysis-ik 插件。  分析器组成 #  该分析器由以下分词器和分词过滤器组成：
 ik_smart 分词器：使用 IK 智能分词模式，倾向于切分出较长的词语 lowercase 分词过滤器：转换为小写  示例 #  POST _analyze { &#34;analyzer&#34;: &#34;ik_smart&#34;, &#34;text&#34;: &#34;中华人民共和国国歌&#34; } 相关指南 #    文本分析基础  "
---


# IK Smart 分析器

`ik_smart` 分析器是为中文智能分词的分析器，使用 IK 分词器的智能模式，会尽量将文本切分为最少的词语。

{{< hint info >}}
需要安装 `analysis-ik` 插件。
{{< /hint >}}

## 分析器组成

该分析器由以下分词器和分词过滤器组成：

- `ik_smart` 分词器：使用 IK 智能分词模式，倾向于切分出较长的词语
- `lowercase` 分词过滤器：转换为小写

## 示例

```json
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "中华人民共和国国歌"
}
```

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

