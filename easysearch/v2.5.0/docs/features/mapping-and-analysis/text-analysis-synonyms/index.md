---
title: "同义词"
date: 0001-01-01
description: "用同义词扩大召回：规则格式、索引时/查询时取舍、分析链顺序与多词同义词陷阱。"
summary: "同义词 #  词干提取通过归并词形变化扩大搜索范围；同义词（Synonyms） 则通过归并“意义接近的不同表达”扩大搜索范围。
例如，查询“英国女王”时，你可能也希望命中包含“英国君主”的文档；用户搜索“美国”时，可能也期望命中包含“美利坚合众国”“USA”等表达的文档。
同义词看起来很简单，但“用对”并不容易：如果把语义跨度过大的词强行绑定，会让结果变得像“随机返回”。经验上，同义词应当 少而精，服务于明确的高价值场景。
基本用法：synonym 过滤器 #  同义词通过 synonym 词元过滤器实现，它会在分析过程中把词项替换或扩展为多个词项：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_synonym_filter&#34;: { &#34;type&#34;: &#34;synonym&#34;, &#34;synonyms&#34;: [ &#34;british,english&#34;, &#34;queen,monarch&#34; ] } }, &#34;analyzer&#34;: { &#34;my_synonyms&#34;: { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;my_synonym_filter&#34; ] } } } } } 同义词通常与原词项处于同一 position，因此短语查询仍能成立（前提是规则与分析链设置得当）。
同义词规则格式 #  权威指南里给出了两种常用格式：
1）等价同义词（逗号分隔） #  jump,leap,hop 含义：遇到其中任意一个词项时，扩展为这一组同义词（等价看待）。
2）定向映射（=&gt;） #  u s a,united states,united states of america =&gt; usa g b,gb,great britain =&gt; britain,england,scotland,wales 含义：把左侧的一组表达统一映射为右侧一个或多个词项。"
---


# 同义词

词干提取通过归并词形变化扩大搜索范围；**同义词（Synonyms）** 则通过归并“意义接近的不同表达”扩大搜索范围。

例如，查询“英国女王”时，你可能也希望命中包含“英国君主”的文档；用户搜索“美国”时，可能也期望命中包含“美利坚合众国”“USA”等表达的文档。

同义词看起来很简单，但“用对”并不容易：如果把语义跨度过大的词强行绑定，会让结果变得像“随机返回”。经验上，同义词应当 **少而精**，服务于明确的高价值场景。

## 基本用法：synonym 过滤器

同义词通过 `synonym` 词元过滤器实现，它会在分析过程中把词项替换或扩展为多个词项：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "british,english",
            "queen,monarch"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  }
}
```

同义词通常与原词项处于**同一 position**，因此短语查询仍能成立（前提是规则与分析链设置得当）。

## 同义词规则格式

权威指南里给出了两种常用格式：

### 1）等价同义词（逗号分隔）

```text
jump,leap,hop
```

含义：遇到其中任意一个词项时，扩展为这一组同义词（等价看待）。

### 2）定向映射（=>）

```text
u s a,united states,united states of america => usa
g b,gb,great britain => britain,england,scotland,wales
```

含义：把左侧的一组表达统一映射为右侧一个或多个词项。

当多条规则存在重叠时，通常会倾向于**最长匹配**，避免短规则截断长短语导致的意外扩展。

## 索引时扩展 vs 查询时扩展

同义词既可以在索引时应用，也可以在查询时应用。权威指南给出的权衡非常实用：

- **索引时扩展**
  - 优点：查询更快/更简单（索引里已经包含扩展后的词项）
  - 缺点：索引会变大；同义词策略变更往往需要重建索引才能完全生效
  - 相关性：不同同义词可能被迫共享 IDF 权重（通用词可能被“抬高”）
- **查询时扩展**
  - 优点：规则可快速迭代，通常无需重建索引
  - 缺点：查询会被重写成更复杂的形式，可能影响性能
  - 相关性：各词项的 IDF 更贴近原始统计

实践建议：

- **稳定、变化极少**的词汇（例如确定的缩写/标准化写法），可考虑索引时扩展
- **业务策略驱动、经常调整**的词汇（品牌/类目/运营口径），更适合查询时扩展

## 同义词与分析链：顺序决定“能否匹配”

同义词过滤器只能看到它前面的 tokenizer/filters 输出，而看不到原始字符串。权威指南用 `U.S.A.` 举了一个典型例子：

```text
原始文本: "U.S.A."
standard tokenizer → (U),(S),(A)
lowercase filter   → (u),(s),(a)
synonym filter     → (usa)
```

因此，如果你在同义词表里写 `U.S.A.`，它永远匹配不到，因为在到达 `synonym` 过滤器时，点号已经不见了、大小写也被归一化了。

同样地，若你同时使用词干提取：

- 把 `synonym` 放在词干提取 **之前**：你可能需要枚举各种词形变化（规则更长更难维护）
- 把 `synonym` 放在词干提取 **之后**：你可以只针对“词干形态”写规则（通常更简洁）

## 大小写敏感的同义词

同义词通常放在 `lowercase` 之后，规则也就默认全是小写。但某些缩写/专名需要区分大小写语义（如 `CAT` 扫描 vs `cat` 动物）。

此时可以考虑：

- 把“大小写敏感的同义词过滤器”放在 `lowercase` 之前（代价是规则需要枚举不同大小写）
- 或者拆分为两套过滤器：一套大小写敏感，一套大小写不敏感（配置会变复杂，需要用 `_analyze` 反复验证）

## 多词同义词：短语查询的经典坑

多词同义词（例如 `united states of america` ⇔ `usa`）会显著影响 position，从而让短语查询出现“诡异匹配”或“本该匹配却不匹配”。

权威指南的结论非常明确：想让短语查询更可靠，倾向于用 **简单收缩**（把多词短语映射为单个词项）：

```text
united states,u s a,united states of america => usa
```

代价是：你无法再用 `united` 或 `states` 去匹配到被收缩后的表达（通常需要额外字段/额外分析链来兼顾）。

## 符号同义词：先做字符映射，再分词

符号/表情往往会在分词阶段被当作标点丢弃，但在某些业务里它们很关键（例如情绪分析）。权威指南建议用 `mapping` 字符过滤器在分词前把符号替换成“可索引的词”：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [
            ":)=>emoticon_happy",
            ":(=>emoticon_sad"
          ]
        }
      },
      "analyzer": {
        "my_emoticons": {
          "char_filter": [ "emoticons" ],
          "tokenizer": "standard",
          "filter": [ "lowercase" ]
        }
      }
    }
  }
}
```

## 实务建议

- 同义词要可控：从少量高价值规则开始，配合真实查询日志回放评估噪声
- 规则版本化：支持回滚，避免一次改动把线上搜索体验“打碎”
- 关注分析链顺序：同义词要匹配的是“分词/归一化后的词项”，不是原始字符串
- 多词同义词优先用“收缩”保证短语查询可靠；需要兼顾精确语义时用多字段/多分析链

## 小结

- 同义词能扩大召回，但用不好会降低搜索可解释性
- 两种核心规则：等价扩展（`,`）与定向映射（`=>`）
- 索引时/查询时各有取舍：性能、灵活性、相关性都会受影响
- 分析链顺序决定规则是否能命中；多词同义词要特别小心短语查询与高亮

下一步可以继续阅读：

- [模糊匹配](./text-analysis-fuzzy.md)
- [停用词](./text-analysis-stopwords.md)
- [相关性基础](../fulltext-search/relevance/scoring-basics.md)

## 参考手册（API 与参数）

- [同义词过滤器（功能手册）]({{< relref "docs/features/mapping-and-analysis/text-analysis/token-filters/synonym.md" >}})
- [同义词图过滤器（功能手册）]({{< relref "docs/features/mapping-and-analysis/text-analysis/token-filters/synonym-graph.md" >}})

