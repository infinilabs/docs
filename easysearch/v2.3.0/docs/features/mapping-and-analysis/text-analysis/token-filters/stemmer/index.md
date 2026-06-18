---
title: "词干分词过滤器（Stemmer）"
date: 0001-01-01
summary: "Stemmer 分词过滤器 #  stemmer 分词过滤器会将单词缩减为其词根或基本形式（也称为词干 stem）。
相关指南（先读这些） #    词干提取  文本分析基础  参数说明 #  词干提取分词过滤器可以通过一个 language（语言）参数进行配置，该参数接受以下值：
 阿拉伯语：arabic 亚美尼亚语：armenian 巴斯克语：basque 孟加拉语：bengali 巴西葡萄牙语：brazilian 保加利亚语：bulgarian 加泰罗尼亚语：catalan 捷克语：czech 丹麦语：danish 荷兰语：dutch、dutch_kp 英语：english（默认）、light_english、lovins、minimal_english、porter2、possessive_english 爱沙尼亚语：estonian 芬兰语：finnish、light_finnish 法语：light_french、french、minimal_french 加利西亚语：galician、minimal_galician（仅复数处理步骤） 德语：light_german、german、german2、minimal_german 希腊语：greek 印地语：hindi 匈牙利语：hungarian、light_hungarian 印尼语：indonesian 爱尔兰语：irish 意大利语：light_italian、italian 库尔德语（索拉尼方言）：sorani 拉脱维亚语：latvian 立陶宛语：lithuanian 挪威语（书面挪威语）：norwegian、light_norwegian、minimal_norwegian 挪威语（新挪威语）：light_nynorsk、minimal_nynorsk 葡萄牙语：light_portuguese、minimal_portuguese、portuguese、portuguese_rslp 罗马尼亚语：romanian 俄语：russian、light_russian 西班牙语：light_spanish、spanish 瑞典语：swedish、light_swedish 土耳其语：turkish   你也可以使用 name 参数作为 language 参数的别名。如果两个参数都被设置，name 参数将被忽略。
 参考样例 #  以下示例请求创建了一个名为 my-stemmer-index 的新索引，并配置了一个带有词干提取过滤器的分词器。"
---


# Stemmer 分词过滤器

`stemmer` 分词过滤器会将单词缩减为其词根或基本形式（也称为词干 `stem`）。

## 相关指南（先读这些）

- [词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 参数说明

词干提取分词过滤器可以通过一个 `language`（语言）参数进行配置，该参数接受以下值：

- **阿拉伯语**：`arabic`
- **亚美尼亚语**：`armenian`
- **巴斯克语**：`basque`
- **孟加拉语**：`bengali`
- **巴西葡萄牙语**：`brazilian`
- **保加利亚语**：`bulgarian`
- **加泰罗尼亚语**：`catalan`
- **捷克语**：`czech`
- **丹麦语**：`danish`
- **荷兰语**：`dutch`、`dutch_kp`
- **英语**：`english`（默认）、`light_english`、`lovins`、`minimal_english`、`porter2`、`possessive_english`
- **爱沙尼亚语**：`estonian`
- **芬兰语**：`finnish`、`light_finnish`
- **法语**：`light_french`、`french`、`minimal_french`
- **加利西亚语**：`galician`、`minimal_galician`（仅复数处理步骤）
- **德语**：`light_german`、`german`、`german2`、`minimal_german`
- **希腊语**：`greek`
- **印地语**：`hindi`
- **匈牙利语**：`hungarian`、`light_hungarian`
- **印尼语**：`indonesian`
- **爱尔兰语**：`irish`
- **意大利语**：`light_italian`、`italian`
- **库尔德语（索拉尼方言）**：`sorani`
- **拉脱维亚语**：`latvian`
- **立陶宛语**：`lithuanian`
- **挪威语（书面挪威语）**：`norwegian`、`light_norwegian`、`minimal_norwegian`
- **挪威语（新挪威语）**：`light_nynorsk`、`minimal_nynorsk`
- **葡萄牙语**：`light_portuguese`、`minimal_portuguese`、`portuguese`、`portuguese_rslp`
- **罗马尼亚语**：`romanian`
- **俄语**：`russian`、`light_russian`
- **西班牙语**：`light_spanish`、`spanish`
- **瑞典语**：`swedish`、`light_swedish`
- **土耳其语**：`turkish`

> 你也可以使用 `name` 参数作为 `language` 参数的别名。如果两个参数都被设置，`name` 参数将被忽略。

## 参考样例

以下示例请求创建了一个名为 `my-stemmer-index` 的新索引，并配置了一个带有词干提取过滤器的分词器。

```
PUT /my-stemmer-index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_english_stemmer": {
          "type": "stemmer",
          "language": "english"
        }
      },
      "analyzer": {
        "my_stemmer_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_english_stemmer"
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
GET /my-stemmer-index/_analyze
{
  "analyzer": "my_stemmer_analyzer",
  "text": "running runs"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "run",
      "start_offset": 0,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "run",
      "start_offset": 8,
      "end_offset": 12,
      "type": "<ALPHANUM>",
      "position": 1
    }
  ]
}
```

