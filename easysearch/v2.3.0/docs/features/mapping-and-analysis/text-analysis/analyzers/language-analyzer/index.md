---
title: "语言分析器（Language）"
date: 0001-01-01
summary: "Language 分析器 #  Easysearch 内置了 34 种语言专用分析器，每种都针对该语言的停用词、词干提取和字符归一化进行了优化。使用时只需将 analyzer 设置为对应的语言名称即可。
支持的语言列表 #     语言 分析器名称 特点      阿拉伯语 arabic 阿拉伯语归一化 + 词干提取    亚美尼亚语 armenian Snowball 词干提取    巴斯克语 basque Snowball 词干提取    孟加拉语 bengali 印度语系归一化 + 词干提取    巴西葡萄牙语 brazilian 巴西葡萄牙语词干提取    保加利亚语 bulgarian 保加利亚语词干提取    加泰罗尼亚语 catalan 省音处理 + Snowball 词干提取    捷克语 czech 捷克语词干提取    丹麦语 danish Snowball 词干提取    荷兰语 dutch 词干覆盖字典 + Snowball    英语 english 所有格处理 + Porter 词干提取    爱沙尼亚语 estonian Snowball 词干提取    芬兰语 finnish Snowball 词干提取    法语 french 省音处理 + 轻量词干提取    加利西亚语 galician 加利西亚语词干提取    德语 german 字符归一化 + 轻量词干提取    希腊语 greek 希腊语专用小写 + 词干提取    印地语 hindi 印度语系归一化 + 词干提取    匈牙利语 hungarian Snowball 词干提取    印度尼西亚语 indonesian 印度尼西亚语词干提取    爱尔兰语 irish 专用小写 + 双重停用词过滤    意大利语 italian 省音处理 + 轻量词干提取    拉脱维亚语 latvian 拉脱维亚语词干提取    立陶宛语 lithuanian Snowball 词干提取    挪威语 norwegian Snowball 词干提取    波斯语 persian 字符过滤 + 阿拉伯语/波斯语归一化    波兰语 polish 波兰语词干提取    葡萄牙语 portuguese 轻量词干提取    罗马尼亚语 romanian 字符归一化 + Snowball 词干提取    俄语 russian Snowball 词干提取    索拉尼语 sorani 索拉尼语归一化 + 词干提取    西班牙语 spanish 轻量词干提取    瑞典语 swedish Snowball 词干提取    泰语 thai Java BreakIterator 泰语分词    土耳其语 turkish 专用小写（İ/I）+ Snowball    使用方式 #  在映射中指定对应语言名称即可："
---


# Language 分析器

Easysearch 内置了 34 种语言专用分析器，每种都针对该语言的停用词、词干提取和字符归一化进行了优化。使用时只需将 `analyzer` 设置为对应的语言名称即可。

## 支持的语言列表

| 语言 | 分析器名称 | 特点 |
|------|-----------|------|
| [阿拉伯语]({{< relref "./arabic-analyzer" >}}) | `arabic` | 阿拉伯语归一化 + 词干提取 |
| [亚美尼亚语]({{< relref "./armenian-analyzer" >}}) | `armenian` | Snowball 词干提取 |
| [巴斯克语]({{< relref "./basque-analyzer" >}}) | `basque` | Snowball 词干提取 |
| [孟加拉语]({{< relref "./bengali-analyzer" >}}) | `bengali` | 印度语系归一化 + 词干提取 |
| [巴西葡萄牙语]({{< relref "./brazilian-analyzer" >}}) | `brazilian` | 巴西葡萄牙语词干提取 |
| [保加利亚语]({{< relref "./bulgarian-analyzer" >}}) | `bulgarian` | 保加利亚语词干提取 |
| [加泰罗尼亚语]({{< relref "./catalan-analyzer" >}}) | `catalan` | 省音处理 + Snowball 词干提取 |
| [捷克语]({{< relref "./czech-analyzer" >}}) | `czech` | 捷克语词干提取 |
| [丹麦语]({{< relref "./danish-analyzer" >}}) | `danish` | Snowball 词干提取 |
| [荷兰语]({{< relref "./dutch-analyzer" >}}) | `dutch` | 词干覆盖字典 + Snowball |
| [英语]({{< relref "./english-analyzer" >}}) | `english` | 所有格处理 + Porter 词干提取 |
| [爱沙尼亚语]({{< relref "./estonian-analyzer" >}}) | `estonian` | Snowball 词干提取 |
| [芬兰语]({{< relref "./finnish-analyzer" >}}) | `finnish` | Snowball 词干提取 |
| [法语]({{< relref "./french-analyzer" >}}) | `french` | 省音处理 + 轻量词干提取 |
| [加利西亚语]({{< relref "./galician-analyzer" >}}) | `galician` | 加利西亚语词干提取 |
| [德语]({{< relref "./german-analyzer" >}}) | `german` | 字符归一化 + 轻量词干提取 |
| [希腊语]({{< relref "./greek-analyzer" >}}) | `greek` | 希腊语专用小写 + 词干提取 |
| [印地语]({{< relref "./hindi-analyzer" >}}) | `hindi` | 印度语系归一化 + 词干提取 |
| [匈牙利语]({{< relref "./hungarian-analyzer" >}}) | `hungarian` | Snowball 词干提取 |
| [印度尼西亚语]({{< relref "./indonesian-analyzer" >}}) | `indonesian` | 印度尼西亚语词干提取 |
| [爱尔兰语]({{< relref "./irish-analyzer" >}}) | `irish` | 专用小写 + 双重停用词过滤 |
| [意大利语]({{< relref "./italian-analyzer" >}}) | `italian` | 省音处理 + 轻量词干提取 |
| [拉脱维亚语]({{< relref "./latvian-analyzer" >}}) | `latvian` | 拉脱维亚语词干提取 |
| [立陶宛语]({{< relref "./lithuanian-analyzer" >}}) | `lithuanian` | Snowball 词干提取 |
| [挪威语]({{< relref "./norwegian-analyzer" >}}) | `norwegian` | Snowball 词干提取 |
| [波斯语]({{< relref "./persian-analyzer" >}}) | `persian` | 字符过滤 + 阿拉伯语/波斯语归一化 |
| [波兰语]({{< relref "./polish-analyzer" >}}) | `polish` | 波兰语词干提取 |
| [葡萄牙语]({{< relref "./portuguese-analyzer" >}}) | `portuguese` | 轻量词干提取 |
| [罗马尼亚语]({{< relref "./romanian-analyzer" >}}) | `romanian` | 字符归一化 + Snowball 词干提取 |
| [俄语]({{< relref "./russian-analyzer" >}}) | `russian` | Snowball 词干提取 |
| [索拉尼语]({{< relref "./sorani-analyzer" >}}) | `sorani` | 索拉尼语归一化 + 词干提取 |
| [西班牙语]({{< relref "./spanish-analyzer" >}}) | `spanish` | 轻量词干提取 |
| [瑞典语]({{< relref "./swedish-analyzer" >}}) | `swedish` | Snowball 词干提取 |
| [泰语]({{< relref "./thai-analyzer" >}}) | `thai` | Java BreakIterator 泰语分词 |
| [土耳其语]({{< relref "./turkish-analyzer" >}}) | `turkish` | 专用小写（İ/I）+ Snowball |

## 使用方式

在映射中指定对应语言名称即可：

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "french"
      }
    }
  }
}
```

也可以设置为索引的默认分析器：

```json
PUT my-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default": {
          "type": "french"
        }
      }
    }
  }
}
```

## 通用参数

所有语言分析器都支持以下参数（部分语言除外）：

| 参数 | 说明 |
|------|------|
| `stopwords` | 自定义停用词列表，默认为对应语言的内置停用词 |
| `stem_exclusion` | 不进行词干提取的词语列表 |

## 相关指南

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
```

