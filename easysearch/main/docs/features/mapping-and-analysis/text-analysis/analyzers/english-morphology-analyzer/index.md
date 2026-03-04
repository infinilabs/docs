---
title: "英语形态分析器（English Morphology）"
date: 0001-01-01
summary: "English Morphology 分析器 #  english_morphology 分析器专为处理复杂的英语文本而设计。与仅执行简单算法剪裁的常规分析器不同，它基于词形还原（Lemmatization）技术，能够精准识别英语词汇的形态变化，并将其还原为词典中的标准原型。
这确保了用户在搜索单词的不同形态（如动词时态 ran/running、名词单复数 foxes/fox、或不规则变化 feet/foot）时，能够实现精准的跨形态匹配。
该分析器由以下分词器和分词过滤器组成：
 standard 分词器：去除大部分标点符号，并依据空格和其他常见分隔符对文本进行分割。 lowercase 分词过滤器：将所有词元转换为小写，以确保匹配时不区分大小写。 english_morphology 分词过滤器：执行英语词汇的形态分析，将动词的时态、形容词的比较级以及名词的复数形式映射到其唯一的语义原型。  相关指南（先读这些） #    文本分析：词干提取  文本分析基础  安装 #  英语形态分词器包含在Morphological Analysis插件中。此插件已包含在Easysearch的bundle包中。
analysis-morphology插件安装命令如下：
bin/easysearch-plugin install analysis-morphology 参考样例 #  以下命令创建一个名为 my_morphology_index 的索引，并为 my_field 字段配置俄语形态分词器的索引：
PUT /my_morphology_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;english_morphology&#34; } } } } 配置自定义分词器 #  在生产环境中，为了兼顾性能和准确度，建议定义一个包含小写化、形态还原和停用词过滤的自定义分析器：
PUT /my_custom_morphology_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;english_morphology&#34;, &#34;stop&#34; ] } } } } } 产生的词元 #  通过形态分析，不同的词形会被索引为相同的词元。"
---


# English Morphology 分析器

`english_morphology` 分析器专为处理复杂的英语文本而设计。与仅执行简单算法剪裁的常规分析器不同，它基于词形还原（Lemmatization）技术，能够精准识别英语词汇的形态变化，并将其还原为词典中的标准原型。

这确保了用户在搜索单词的不同形态（如动词时态 ran/running、名词单复数 foxes/fox、或不规则变化 feet/foot）时，能够实现精准的跨形态匹配。

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：去除大部分标点符号，并依据空格和其他常见分隔符对文本进行分割。
- `lowercase` 分词过滤器：将所有词元转换为小写，以确保匹配时不区分大小写。
- `english_morphology` 分词过滤器：执行英语词汇的形态分析，将动词的时态、形容词的比较级以及名词的复数形式映射到其唯一的语义原型。

## 相关指南（先读这些）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 安装

英语形态分词器包含在Morphological Analysis插件中。此插件已包含在Easysearch的bundle包中。

analysis-morphology插件安装命令如下：

```
bin/easysearch-plugin install analysis-morphology
```

## 参考样例

以下命令创建一个名为 `my_morphology_index` 的索引，并为 `my_field` 字段配置俄语形态分词器的索引：

```
PUT /my_morphology_index
{
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "english_morphology"
      }
    }
  }
}
```

## 配置自定义分词器

在生产环境中，为了兼顾性能和准确度，建议定义一个包含**小写化**、**形态还原**和**停用词过滤**的自定义分析器：

```
PUT /my_custom_morphology_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "english_morphology",
            "stop"
          ]
        }
      }
    }
  }
}
```

## 产生的词元

通过形态分析，不同的词形会被索引为相同的词元。

以下请求用来检查分词器生成的词元：

```
POST /my_morphology_index/_analyze
{
  "analyzer": "english_morphology",
  "text": "A runner was running."
}
```

返回内容中包含了产生的词元：

```
{
  "tokens": [
    { "token": "a", "start_offset": 0, "end_offset": 1, "type": "<ALPHANUM>", "position": 0 },
    { "token": "run", "start_offset": 2, "end_offset": 8, "type": "<ALPHANUM>", "position": 1 },
    { "token": "runner", "start_offset": 2, "end_offset": 8, "type": "<ALPHANUM>", "position": 1 },
    { "token": "be", "start_offset": 9, "end_offset": 12, "type": "<ALPHANUM>", "position": 2 },
    { "token": "running", "start_offset": 13, "end_offset": 20, "type": "<ALPHANUM>", "position": 3 },
    { "token": "run", "start_offset": 13, "end_offset": 20, "type": "<ALPHANUM>", "position": 3 }
  ]
}
```

### 返回结果分析

英语形态分词器能够识别单词的派生关系及动词时态，并同时索引多个相关的原型词元：

| 原始单词 | 产生的词元(原型) | 说明                                         |
|---------|---------------|----------------------------------------------|
| runner  | runner, run   | 派生名词还原：既保留独立名词，又关联到动词词根 run。 |
| was     | be            | 不规则动词还原：将助动词 was 还原为原形 be。       |
| running | running, run  | 多重形态识别：索引独立名词形式及动词 run 的分词形式。|

## 与常规分词器的区别

在 Elasticsearch 中，处理英语通常有两种主流方式：一种是基于 **词干提取（Stemming）** 的常规分词器（如 english analyzer），
另一种是基于 **词形还原（Lemmatization）** 的形态分词器（如 analysis-morphology 插件）。

| 特性      | 常规分词器 (基于Stemming)                                 | 形态分词器 (基于Lemmatization)                   |
|----------|---------------------------------------------------------|-----------------------------------------------|
| 处理方式  | 启发式剪裁：根据算法(如Porter Stemmer) 剥离后缀(如-ing,-ed,-s)| 词典比对：通过内置词典查找单词的真实词源 (Lemma)     |
| 结果准确性 | 较低。容易产生非单词的词根（如 universal 变为 univers）      | 极高。始终返回具有语义的真实单词 (如saw还原为see或saw)|
| 过度匹配   | 常见。可能会将 organization 和 organ 还原为同一词根造成误报  | 极少。基于语法和词典，严格区分语义                   |
| 性能消耗   | 极低。算法简单，计算速度极快                                | 较高。需要加载和检索词典，内存和 CPU 消耗略大        |

## 演示
具体地展示了英语形态分词器的工作原理和使用方法的脚本，详见[俄语形态分词器](./russian-morphology-analyzer#更多示例)。

