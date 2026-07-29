---
title: "IK最大词分词器（IK Max Word）"
date: 0001-01-01
summary: "IK Max Word 分词器 #  ik_max_word 是 IK 分词器插件提供的细粒度分词模式。
分词方式 #  最大词模式（Max Word）倾向于将文本分成最细粒度的词项，适合对召回率要求高的场景。
示例 #  POST _analyze { &#34;tokenizer&#34;: &#34;ik_max_word&#34;, &#34;text&#34;: &#34;北京市朝阳区建国路1号&#34; } 分析结果 #  [ &#34;北京市&#34;, &#34;北京&#34;, &#34;市&#34;, &#34;朝阳区&#34;, &#34;朝阳&#34;, &#34;区&#34;, &#34;建国路&#34;, &#34;建国&#34;, &#34;路&#34;, &#34;1&#34;, &#34;号&#34; ] 相关指南 #    IK分词器文档  文本分析基础  依赖插件 #   analysis-ik 插件  配置 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_ik_max_word&#34;: { &#34;type&#34;: &#34;ik_max_word&#34; } } } } } "
---


# IK Max Word 分词器

`ik_max_word` 是 IK 分词器插件提供的细粒度分词模式。

## 分词方式

最大词模式（Max Word）倾向于将文本分成最细粒度的词项，适合对召回率要求高的场景。

## 示例

```json
POST _analyze
{
  "tokenizer": "ik_max_word",
  "text": "北京市朝阳区建国路1号"
}
```

## 分析结果

```
[ "北京市", "北京", "市", "朝阳区", "朝阳", "区", "建国路", "建国", "路", "1", "号" ]
```

## 相关指南

- [IK分词器文档]({{< relref "/docs/features/mapping-and-analysis/text-analysis/tokenizers/ik-smart.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 依赖插件

- `analysis-ik` 插件

## 配置

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_ik_max_word": {
          "type": "ik_max_word"
        }
      }
    }
  }
}
```

