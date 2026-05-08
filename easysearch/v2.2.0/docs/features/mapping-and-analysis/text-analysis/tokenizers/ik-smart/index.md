---
title: "IK智能分词器（IK Smart）"
date: 0001-01-01
summary: "IK Smart 分词器 #  ik_smart 是 IK 分词器插件提供的智能分词模式，是最常用的中文分词方式。
##分词方式
智能分词（Smart）模式倾向于将文本分成&quot;人类可读&quot;的粗粒度词项，适合大多数应用场景。
示例 #  POST _analyze { &#34;tokenizer&#34;: &#34;ik_smart&#34;, &#34;text&#34;: &#34;北京市朝阳区建国路1号&#34; } 分析结果 #  [ &#34;北京市&#34;, &#34;朝阳区&#34;, &#34;建国路&#34;, &#34;1&#34;, &#34;号&#34; ] 相关指南 #    IK分词器文档  文本分析基础  依赖插件 #   analysis-ik 插件  配置 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_ik_smart&#34;: { &#34;type&#34;: &#34;ik_smart&#34; } } } } } "
---


# IK Smart 分词器

`ik_smart` 是 IK 分词器插件提供的智能分词模式，是最常用的中文分词方式。

##分词方式

智能分词（Smart）模式倾向于将文本分成"人类可读"的粗粒度词项，适合大多数应用场景。

## 示例

```json
POST _analyze
{
  "tokenizer": "ik_smart",
  "text": "北京市朝阳区建国路1号"
}
```

## 分析结果

```
[ "北京市", "朝阳区", "建国路", "1", "号" ]
```

## 相关指南

- [IK分词器文档]({{< relref "/docs/features/mapping-and-analysis/text-analysis/tokenizers/ik-max-word.md" >}})
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
        "my_ik_smart": {
          "type": "ik_smart"
        }
      }
    }
  }
}
```

