---
title: "ICU 分词器（ICU Tokenizer）"
date: 0001-01-01
summary: "ICU Tokenizer #  icu_tokenizer 分词器使用 ICU 的 Unicode 文本分割算法，对多语言文本（尤其是亚洲语言混合文本）提供比 standard 分词器更好的分词效果。
需要安装 analysis-icu 插件。
示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_icu&#34;: { &#34;type&#34;: &#34;icu_tokenizer&#34; } } } } } 参数 #     参数 说明 默认值     rule_files 自定义 ICU 分词规则文件路径 无    相关指南 #    ICU 分析器  Standard 分词器  文本分析基础  "
---


# ICU Tokenizer

`icu_tokenizer` 分词器使用 ICU 的 Unicode 文本分割算法，对多语言文本（尤其是亚洲语言混合文本）提供比 `standard` 分词器更好的分词效果。

需要安装 `analysis-icu` 插件。

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_icu": {
          "type": "icu_tokenizer"
        }
      }
    }
  }
}
```

## 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `rule_files` | 自定义 ICU 分词规则文件路径 | 无 |

## 相关指南

- [ICU 分析器]({{< relref "../analyzers/icu-analyzer" >}})
- [Standard 分词器]({{< relref "./standard" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

