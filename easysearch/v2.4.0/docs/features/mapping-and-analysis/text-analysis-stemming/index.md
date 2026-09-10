---
title: "词干提取"
date: 0001-01-01
description: "将词形变化收敛到统一词根：词干提取的收益、风险与控制方式。"
summary: "词干提取 #  很多语言的单词都存在“词形变化”（inflection）：同一个概念会换着马甲出现：
 单复数：fox / foxes 时态：pay / paid / paying 性别：waiter / waitress 人称变化：hear / hears  如果只做精确词项匹配，用户搜索 fox 时不会命中只包含 foxes 的文档。词干提取（Stemming） 的目标就是把这些“形式差异”抹平：让相关词项尽量归并到同一个词根上，从而提升召回率。
词干提取会遇到的两类问题 #  词干提取不是“精确科学”，它更像“有原则的近似”。常见两类风险：
 弱提取（Understemming）：相关词没有收敛到同一词干
例如：jumped、jumps → jump，但 jumping → jumpi，导致漏召回。 过度提取（Overstemming）：本应区分的词被合并到同一词干
例如：general 与 generate 都被提取为 gener，导致误召回、降低精度。  这也是为什么“哪个词干器最好”没有统一答案：你的语料、你的用户、你的容错边界都不一样。
词干提取 vs 词形还原（Lemmatization） #   词干提取：规则化“裁剪”词形，得到的词根未必是真实单词（如 jumpi）。
重点是：索引与查询两端一致即可。 词形还原：尝试按“词义/词典形态”归类（如 is/was/am/being → be），通常更复杂、成本更高，需要更多语言知识与上下文。  在多数搜索系统里，词干提取更常作为默认选择：成本低、收益稳。词形还原则更像“精装修”，做得好很香，做不好也可能很贵。
算法词干器（Algorithmic stemmers） #  Easysearch 常用的是算法型词干器：通过一系列规则把词映射到词根，不依赖大词典。你可以把它理解为“规则派”：不查字典，按套路办事。
优点：
 速度快、内存占用小 对“规律变化”的词效果稳定  缺点："
---


# 词干提取

很多语言的单词都存在“词形变化”（inflection）：同一个概念会换着马甲出现：

- **单复数**：`fox` / `foxes`
- **时态**：`pay` / `paid` / `paying`
- **性别**：`waiter` / `waitress`
- **人称变化**：`hear` / `hears`

如果只做精确词项匹配，用户搜索 `fox` 时不会命中只包含 `foxes` 的文档。**词干提取（Stemming）** 的目标就是把这些“形式差异”抹平：让相关词项尽量归并到同一个词根上，从而提升召回率。

## 词干提取会遇到的两类问题

词干提取不是“精确科学”，它更像“有原则的近似”。常见两类风险：

- **弱提取（Understemming）**：相关词没有收敛到同一词干  
  例如：`jumped`、`jumps` → `jump`，但 `jumping` → `jumpi`，导致漏召回。
- **过度提取（Overstemming）**：本应区分的词被合并到同一词干  
  例如：`general` 与 `generate` 都被提取为 `gener`，导致误召回、降低精度。

这也是为什么“哪个词干器最好”没有统一答案：你的语料、你的用户、你的容错边界都不一样。

## 词干提取 vs 词形还原（Lemmatization）

- **词干提取**：规则化“裁剪”词形，得到的词根未必是真实单词（如 `jumpi`）。  
  重点是：索引与查询两端一致即可。
- **词形还原**：尝试按“词义/词典形态”归类（如 `is/was/am/being` → `be`），通常更复杂、成本更高，需要更多语言知识与上下文。

在多数搜索系统里，**词干提取**更常作为默认选择：成本低、收益稳。词形还原则更像“精装修”，做得好很香，做不好也可能很贵。

## 算法词干器（Algorithmic stemmers）

Easysearch 常用的是**算法型词干器**：通过一系列规则把词映射到词根，不依赖大词典。你可以把它理解为“规则派”：不查字典，按套路办事。

优点：

- 速度快、内存占用小
- 对“规律变化”的词效果稳定

缺点：

- 对不规则词形支持有限（如 `mice` / `mouse`）
- 需要控制过度提取带来的噪声

### 示例：在分析链中加入词干器（示意）

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "english_stemmer": {
          "type": "stemmer",
          "language": "english"
        }
      },
      "analyzer": {
        "my_english": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "english_stemmer"
          ]
        }
      }
    }
  }
}
```

如果你发现默认提干“太激进”，可以选择更轻量的变体（例如 `light_english`），减少过度提取的风险（具体可选项以你部署的 Easysearch 为准）。

## 控制词干提取：排除与覆写

开箱即用的词干提取不可能在所有业务里都完美。常见的两种“纠偏”方式：

### 1）阻止某些词被提干（keyword_marker）

如果你希望某些词保持原样（例如业务术语、人名、品牌），可以先用 `keyword_marker` 标记它们，后续词干器会跳过这些词：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "no_stem": {
          "type": "keyword_marker",
          "keywords": [ "skies" ]
        }
      },
      "analyzer": {
        "my_english": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "no_stem",
            "english_stemmer"
          ]
        }
      }
    }
  }
}
```

实践中也可以把排除列表放到文件（`keywords_path`）中统一维护，便于运维更新（不然每次改规则都要改配置，挺累的）。

### 2）自定义词干结果（stemmer_override）

对于不规则变化或特定业务词形，你可以显式指定“原词 => 词根”的覆写规则：

```json
"custom_stem": {
  "type": "stemmer_override",
  "rules": [
    "skies=>sky",
    "mice=>mouse",
    "feet=>foot"
  ]
}
```

关键点：`stemmer_override` 必须放在通用词干器 **之前**，先让“特例”落地，再让“通用规则”接管。

## 不建议：把“原词 + 词干”塞进同一个字段

有一种做法会把“原词”和“提干后的词”放在同一字段同一位置（例如 `foxes,fox`），看起来很省事（一个字段搞定），但通常会带来两个问题：

- **无法区分精确匹配与宽松匹配**：很难在查询时用 boost 清晰地区分“原词命中”和“提干命中”的贡献
- **相关性统计被扭曲**：同一文档里词根出现次数被放大，会影响 TF/IDF 等统计，导致排序不稳定

更推荐的方式是使用 **multi-fields**（一份内容，多条索引路线）：

- `title`：保留更“原始/精确”的分析链
- `title.stemmed`：启用更宽松的词干提取

查询时组合两个字段：原始字段主导排序，提干字段扩大召回并作为补充信号。这样“既要又要”，还更可控。

## 如何选择词干策略（实务）

- **面向终端用户检索**：优先选择更温和/轻量的词干器，避免误召回导致“看起来随机”
- **文档规模较小**：可以稍微更激进一点以提升可搜到的概率
- **文档规模很大**：更温和通常更稳，噪声会被放大得更明显
- **关键术语**：用 `keyword_marker`/自定义规则保护

## 小结

- 词干提取通过归并词形变化提升召回率，但需要警惕弱提取与过度提取
- 算法词干器成本低、速度快，是多数场景的首选
- 通过 `keyword_marker` 与 `stemmer_override` 可以对词干提取做“可控的例外”
- 推荐使用 multi-fields 分离“精确字段”和“宽松字段”，查询时组合使用

下一步可以继续阅读：

- [停用词](./text-analysis-stopwords.md)
- [同义词](./text-analysis-synonyms.md)
- [模糊匹配](./text-analysis-fuzzy.md)

## 参考手册（API 与参数）

- [词干过滤器（功能手册）]({{< relref "docs/features/mapping-and-analysis/text-analysis/token-filters/stemmer.md" >}})
- [Snowball 词干过滤器（功能手册）]({{< relref "docs/features/mapping-and-analysis/text-analysis/token-filters/snowball.md" >}})

