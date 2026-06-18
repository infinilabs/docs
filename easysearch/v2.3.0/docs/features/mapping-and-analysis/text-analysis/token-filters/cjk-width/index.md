---
title: "CJK 宽度分词过滤器（CJK Width）"
date: 0001-01-01
summary: "CJK Width 分词过滤器 #  cjk_width 分词过滤器通过将全角 ASCII 字符转换为其标准（半角）ASCII 等效字符，并将半角片假名字符转换为全角等效字符，来对中文、日文和韩文（CJK）词元进行规范化处理。
相关指南（先读这些） #    文本分析：规范化  文本分析：识别词元  转换全角 ASCII 字符 #  在 CJK 文本中，ASCII 字符（如字母和数字）可能会以全角形式出现，占据两个半角字符的空间。全角 ASCII 字符在东亚排版中通常用于与 CJK 字符的宽度对齐。然而，出于索引和搜索的目的，这些全角字符需要规范化为其标准（半角）ASCII 等效字符。
以下示例说明了 ASCII 字符的规范化过程：
全角: ＡＢＣＤＥ １２３４５ 规范化 (半角): ABCDE 12345 转换半角片假名字符 #  CJK 宽度分词过滤器会将半角片假名字符转换为对应的全角片假名字符，全角片假名是日语文本中使用的标准形式。如下例所示，这种规范化处理对于文本处理和搜索的一致性至关重要：
半角 katakana: ｶﾀｶﾅ 规范化 (全角) katakana: カタカナ 参考样例 #  以下示例请求创建了一个名为 cjk_width_example_index 的新索引，并定义了一个包含 cjk_width 过滤器的分词器：
PUT /cjk_width_example_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;cjk_width_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [&#34;cjk_width&#34;] } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# CJK Width 分词过滤器

`cjk_width` 分词过滤器通过将全角 ASCII 字符转换为其标准（半角）ASCII 等效字符，并将半角片假名字符转换为全角等效字符，来对中文、日文和韩文（CJK）词元进行规范化处理。

## 相关指南（先读这些）

- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

### 转换全角 ASCII 字符

在 CJK 文本中，ASCII 字符（如字母和数字）可能会以全角形式出现，占据两个半角字符的空间。全角 ASCII 字符在东亚排版中通常用于与 CJK 字符的宽度对齐。然而，出于索引和搜索的目的，这些全角字符需要规范化为其标准（半角）ASCII 等效字符。

以下示例说明了 ASCII 字符的规范化过程：

```
        全角:              ＡＢＣＤＥ １２３４５
        规范化 (半角): ABCDE 12345
```

### 转换半角片假名字符

CJK 宽度分词过滤器会将半角片假名字符转换为对应的全角片假名字符，全角片假名是日语文本中使用的标准形式。如下例所示，这种规范化处理对于文本处理和搜索的一致性至关重要：

```
        半角 katakana:          ｶﾀｶﾅ
        规范化 (全角) katakana:  カタカナ
```

## 参考样例

以下示例请求创建了一个名为 `cjk_width_example_index` 的新索引，并定义了一个包含 `cjk_width` 过滤器的分词器：

```
PUT /cjk_width_example_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "cjk_width_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["cjk_width"]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /cjk_width_example_index/_analyze
{
  "analyzer": "cjk_width_analyzer",
  "text": "Ｔｏｋｙｏ 2024 ｶﾀｶﾅ"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "Tokyo",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "2024",
      "start_offset": 6,
      "end_offset": 10,
      "type": "<NUM>",
      "position": 1
    },
    {
      "token": "カタカナ",
      "start_offset": 11,
      "end_offset": 15,
      "type": "<KATAKANA>",
      "position": 2
    }
  ]
}
```

