---
title: "简繁体转换分词器（ST Convert）"
date: 0001-01-01
summary: "ST Convert 分词器 #  stconvert 分词器用于中文简繁体转换，可将简体中文转换为繁体中文，或将繁体中文转换为简体中文。
需要安装 analysis-stconvert 插件。
参数 #     参数 说明 默认值     convert_type 转换类型：s2t（简→繁）或 t2s（繁→简） s2t   keep_both 是否同时保留转换前后的词元 false   delimiter 用于分隔转换结果的分隔符 ,    示例 #  PUT my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;tokenizer&#34;: { &#34;my_stconvert&#34;: { &#34;type&#34;: &#34;stconvert&#34;, &#34;convert_type&#34;: &#34;s2t&#34; } }, &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;tokenizer&#34;: &#34;my_stconvert&#34; } } } } } 相关指南 #    ST Convert 分析器  文本分析基础  "
---


# ST Convert 分词器

`stconvert` 分词器用于中文简繁体转换，可将简体中文转换为繁体中文，或将繁体中文转换为简体中文。

需要安装 `analysis-stconvert` 插件。

## 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `convert_type` | 转换类型：`s2t`（简→繁）或 `t2s`（繁→简） | `s2t` |
| `keep_both` | 是否同时保留转换前后的词元 | `false` |
| `delimiter` | 用于分隔转换结果的分隔符 | `,` |

## 示例

```json
PUT my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_stconvert": {
          "type": "stconvert",
          "convert_type": "s2t"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_stconvert"
        }
      }
    }
  }
}
```

## 相关指南

- [ST Convert 分析器]({{< relref "../analyzers/stconvert-analyzer" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

