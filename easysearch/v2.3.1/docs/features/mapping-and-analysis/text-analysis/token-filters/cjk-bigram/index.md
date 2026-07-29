---
title: "CJK 二元组分词过滤器（CJK Bigram）"
date: 0001-01-01
summary: "CJK Bigram 分词过滤器 #  cjk_bigram 分词过滤器是专门为处理东亚语言（如中文、日文和韩文，简称 CJK）而设计的，这些语言通常不使用空格来分隔单词。二元组是指在词元字符串中两个相邻元素的序列，这些元素可以是字符或单词。对于 CJK 语言来说，二元组有助于近似确定单词边界，并捕捉那些能够传达意义的重要字符对。
相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参数说明 #  CJK 二元分词过滤器可以通过两个参数进行配置：ignore_scripts（忽略字符集）和 output_unigrams（输出一元组）。
ignore_scripts（忽略字符集） #  CJK 二元分词过滤器会忽略所有非 CJK 字符集（如拉丁字母或西里尔字母等书写系统），并且仅将 CJK 文本分词为二元组。可使用此选项指定要忽略的 CJK 字符集。该选项有以下有效值：
 han（汉字）：汉字字符集处理汉字。汉字是用于中国、日本和韩国书面语言的意符文字。该过滤器可帮助处理文本相关任务，例如对用中文、日文汉字或韩文汉字书写的文本进行分词、规范化或词干提取。 hangul（韩文）：韩文（谚文）字符集处理韩文字符，这些字符是韩语所特有的，不存在于其他东亚字符集中。 hiragana（平假名）：平假名字符集处理平假名，平假名是日语书写系统中使用的两种音节文字之一。平假名通常用于日语的本土词汇、语法元素以及某些形式的标点符号。 katakana（片假名）：片假名字符集处理片假名，片假名是日语的另一种音节文字。片假名主要用于外来借词、拟声词、科学名称以及某些日语词汇。  output_unigrams（输出一元组） #  当此选项设置为 true 时，会同时输出一元组（单个字符）和二元组。默认值为 false。
参考样例 #  以下示例请求创建了一个名为 devanagari_example_index 的新索引，并定义了一个分词器，该分词器使用了 cjk_bigram_filter 过滤器，且将 ignored_scripts 参数设置为 katakana（片假名）：
PUT /cjk_bigram_example { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;cjk_bigrams_no_katakana&#34;: { &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;cjk_bigrams_no_katakana_filter&#34; ] } }, &#34;filter&#34;: { &#34;cjk_bigrams_no_katakana_filter&#34;: { &#34;type&#34;: &#34;cjk_bigram&#34;, &#34;ignored_scripts&#34;: [ &#34;katakana&#34; ], &#34;output_unigrams&#34;: true } } } } } 产生的词元 #  使用以下请求来检查使用该分词器生成的词元："
---


# CJK Bigram 分词过滤器

`cjk_bigram` 分词过滤器是专门为处理东亚语言（如中文、日文和韩文，简称 CJK）而设计的，这些语言通常不使用空格来分隔单词。二元组是指在词元字符串中两个相邻元素的序列，这些元素可以是字符或单词。对于 CJK 语言来说，二元组有助于近似确定单词边界，并捕捉那些能够传达意义的重要字符对。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

CJK 二元分词过滤器可以通过两个参数进行配置：`ignore_scripts`（忽略字符集）和 `output_unigrams`（输出一元组）。

### `ignore_scripts`（忽略字符集）

CJK 二元分词过滤器会忽略所有非 CJK 字符集（如拉丁字母或西里尔字母等书写系统），并且仅将 CJK 文本分词为二元组。可使用此选项指定要忽略的 CJK 字符集。该选项有以下有效值：

- **han（汉字）**：汉字字符集处理汉字。汉字是用于中国、日本和韩国书面语言的意符文字。该过滤器可帮助处理文本相关任务，例如对用中文、日文汉字或韩文汉字书写的文本进行分词、规范化或词干提取。
- **hangul（韩文）**：韩文（谚文）字符集处理韩文字符，这些字符是韩语所特有的，不存在于其他东亚字符集中。
- **hiragana（平假名）**：平假名字符集处理平假名，平假名是日语书写系统中使用的两种音节文字之一。平假名通常用于日语的本土词汇、语法元素以及某些形式的标点符号。
- **katakana（片假名）**：片假名字符集处理片假名，片假名是日语的另一种音节文字。片假名主要用于外来借词、拟声词、科学名称以及某些日语词汇。

### `output_unigrams`（输出一元组）

当此选项设置为 `true` 时，会同时输出一元组（单个字符）和二元组。默认值为 `false`。

## 参考样例

以下示例请求创建了一个名为 `devanagari_example_index` 的新索引，并定义了一个分词器，该分词器使用了 `cjk_bigram_filter` 过滤器，且将 `ignored_scripts` 参数设置为 `katakana`（片假名）：

```
PUT /cjk_bigram_example
{
  "settings": {
    "analysis": {
      "analyzer": {
        "cjk_bigrams_no_katakana": {
          "tokenizer": "standard",
          "filter": [ "cjk_bigrams_no_katakana_filter" ]
        }
      },
      "filter": {
        "cjk_bigrams_no_katakana_filter": {
          "type": "cjk_bigram",
          "ignored_scripts": [
            "katakana"
          ],
          "output_unigrams": true
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /cjk_bigram_example/_analyze
{
  "analyzer": "cjk_bigrams_no_katakana",
  "text": "東京タワーに行く"
}
```

测试文本：“東京タワーに行く”

```
東京 (Kanji for "Tokyo")
タワー (Katakana for "Tower")
に行く (Hiragana and Kanji for "go to")
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "東",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<SINGLE>",
      "position": 0
    },
    {
      "token": "東京",
      "start_offset": 0,
      "end_offset": 2,
      "type": "<DOUBLE>",
      "position": 0,
      "positionLength": 2
    },
    {
      "token": "京",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<SINGLE>",
      "position": 1
    },
    {
      "token": "タワー",
      "start_offset": 2,
      "end_offset": 5,
      "type": "<KATAKANA>",
      "position": 2
    },
    {
      "token": "に",
      "start_offset": 5,
      "end_offset": 6,
      "type": "<SINGLE>",
      "position": 3
    },
    {
      "token": "に行",
      "start_offset": 5,
      "end_offset": 7,
      "type": "<DOUBLE>",
      "position": 3,
      "positionLength": 2
    },
    {
      "token": "行",
      "start_offset": 6,
      "end_offset": 7,
      "type": "<SINGLE>",
      "position": 4
    },
    {
      "token": "行く",
      "start_offset": 6,
      "end_offset": 8,
      "type": "<DOUBLE>",
      "position": 4,
      "positionLength": 2
    },
    {
      "token": "く",
      "start_offset": 7,
      "end_offset": 8,
      "type": "<SINGLE>",
      "position": 5
    }
  ]
}
```

