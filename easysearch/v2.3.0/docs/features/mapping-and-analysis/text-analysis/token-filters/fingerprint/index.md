---
title: "指纹分词过滤器（Fingerprint）"
date: 0001-01-01
summary: "Fingerprint 分词过滤器 #  fingerprint 分词过滤器用于对文本进行标准化和去重处理。当文本处理的唯一性或者一致性至关重要时，这个过滤器特别有用。fingerprint 分词过滤器通过以下步骤处理文本以实现这一目的：
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元   小写转换：将所有文本转换为小写。 分词：把文本拆分成词元。 排序：按字母顺序排列这些词元。 去重：消除重复的词元。 合并词元：将这些词元合并成一个字符串，通常使用空格或其他指定的分隔符进行连接。  参数说明 #  指纹分词过滤器可以使用以下两个参数进行配置。
   参数 必需/可选 数据类型 描述     max_output_size 可选 整数 限制生成的指纹字符串的长度。如果拼接后的字符串超过了 max_output_size，过滤器将不会产生任何输出，从而得到一个空词元。默认值是 255   separator 可选 字符串 定义在词元排序和去重后，用于将词元连接成单个字符串。默认是空格（&quot; &quot;）。    参考样例 #  以下示例请求创建了一个名为 my_index 的新索引，并配置了一个带有指纹分词过滤器的分词器：
PUT /my_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;filter&#34;: { &#34;my_fingerprint&#34;: { &#34;type&#34;: &#34;fingerprint&#34;, &#34;max_output_size&#34;: 200, &#34;separator&#34;: &#34;-&#34; } }, &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;my_fingerprint&#34; ] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# Fingerprint 分词过滤器

`fingerprint` 分词过滤器用于对文本进行标准化和去重处理。当文本处理的唯一性或者一致性至关重要时，这个过滤器特别有用。`fingerprint` 分词过滤器通过以下步骤处理文本以实现这一目的：

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

1. **小写转换**：将所有文本转换为小写。
2. **分词**：把文本拆分成词元。
3. **排序**：按字母顺序排列这些词元。
4. **去重**：消除重复的词元。
5. **合并词元**：将这些词元合并成一个字符串，通常使用空格或其他指定的分隔符进行连接。

## 参数说明

指纹分词过滤器可以使用以下两个参数进行配置。

| 参数              | 必需/可选 | 数据类型 | 描述                                                                                                                                 |
| ----------------- | --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `max_output_size` | 可选      | 整数     | 限制生成的指纹字符串的长度。如果拼接后的字符串超过了 `max_output_size`，过滤器将不会产生任何输出，从而得到一个空词元。默认值是 `255` |
| `separator`       | 可选      | 字符串   | 定义在词元排序和去重后，用于将词元连接成单个字符串。默认是空格（`" "`）。                                                            |

## 参考样例

以下示例请求创建了一个名为 `my_index` 的新索引，并配置了一个带有指纹分词过滤器的分词器：

```
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_fingerprint": {
          "type": "fingerprint",
          "max_output_size": 200,
          "separator": "-"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_fingerprint"
          ]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Easysearch is a powerful search engine that scales easily"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "a-easily-engine-is-easysearch-powerful-scales-search-that",
      "start_offset": 0,
      "end_offset": 57,
      "type": "fingerprint",
      "position": 0
    }
  ]
}
```

