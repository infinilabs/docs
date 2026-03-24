---
title: "单词分隔图分词过滤器（Word Delimiter Graph）"
date: 0001-01-01
summary: "Word Delimiter Graph 分词过滤器 #  word_delimiter_graph 分词过滤器用于依据预定义的字符对词元进行拆分，还能基于可定制的规则对词元进行可选的规范化处理。
 提示：word_delimiter_graph 分词过滤器可用于去除零件编号或产品 ID 这类复杂标识符中的标点符号。在这种情况下，它最好与 keyword 分词器搭配使用。对于带有连字符的单词，建议使用 synonym_graph 分词过滤器而非 word_delimiter_graph 分词过滤器，因为用户在搜索这些词时，往往既会搜索带连字符的形式，也会搜索不带连字符的形式。
 相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  默认情况下，该过滤器会应用以下规则：
   描述 输入 输出     将非字母数字字符视为分隔符 ultra-fast ultra, fast   去除词元开头或结尾的分隔符 Z99++'Decoder' Z99, Decoder   当字母大小写发生转换时拆分词元 Easysearch Easy, search   当字母和数字之间发生转换时拆分词元 T1000 T, 1000   去除词元末尾的所有格形式（&lsquo;s） John's John     重要的是，不要将此过滤器与会去除标点符号的分词器（如标准分词器）一起使用。这样做可能会导致词元无法正确拆分，并且会影响诸如 catenate_all 或 preserve_original 之类的选项。我们建议将此过滤器与关键字(keyword)分词器或空白(whitespace)分词器搭配使用。"
---


# Word Delimiter Graph 分词过滤器

`word_delimiter_graph` 分词过滤器用于依据预定义的字符对词元进行拆分，还能基于可定制的规则对词元进行可选的规范化处理。

> **提示**：`word_delimiter_graph` 分词过滤器可用于去除零件编号或产品 ID 这类复杂标识符中的标点符号。在这种情况下，它最好与 `keyword` 分词器搭配使用。对于带有连字符的单词，建议使用 `synonym_graph` 分词过滤器而非 `word_delimiter_graph` 分词过滤器，因为用户在搜索这些词时，往往既会搜索带连字符的形式，也会搜索不带连字符的形式。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

默认情况下，该过滤器会应用以下规则：

| 描述                               | 输入             | 输出             |
| ---------------------------------- | ---------------- | ---------------- |
| 将非字母数字字符视为分隔符         | `ultra-fast`     | `ultra`, `fast`  |
| 去除词元开头或结尾的分隔符         | `Z99++'Decoder'` | `Z99`, `Decoder` |
| 当字母大小写发生转换时拆分词元     | `Easysearch`     | `Easy`, `search` |
| 当字母和数字之间发生转换时拆分词元 | `T1000`          | `T`, `1000`      |
| 去除词元末尾的所有格形式（'s）     | `John's`         | `John`           |

> 重要的是，不要将此过滤器与会去除标点符号的分词器（如标准分词器）一起使用。这样做可能会导致词元无法正确拆分，并且会影响诸如 `catenate_all` 或 `preserve_original` 之类的选项。我们建议将此过滤器与关键字(`keyword`)分词器或空白(`whitespace`)分词器搭配使用。

## 参数说明

你可以使用以下参数来配置单词分隔符图分词过滤器。

| 参数                      | 必需/可选 | 数据类型   | 描述                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------- | --------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `adjust_offsets`          | 可选      | 布尔值     | 决定是否要为拆分或连接后的词元重新计算词元偏移量。若为 `true`，过滤器会调整词元偏移量，以准确呈现词元在词元流中的位置。这种调整能确保词元在文本中的位置与处理后的修改形式相匹配，这对高亮显示或短语查询等应用特别有用。若为 `false`，偏移量保持不变，这可能会导致处理后的词元映射回原始文本位置时出现错位。如果你的分词器使用了像 `trim` 这类会改变词元长度但不改变偏移量的过滤器，建议将此参数设为 `false`。默认值为 `true`。 |
| `catenate_all`            | 可选      | 布尔值     | 从一系列字母数字部分生成连接后的词元。例如，`quick-fast-200` 会变成 `[ quickfast200, quick, fast, 200 ]`。默认值为 `false`。                                                                                                                                                                                                                                                                                                   |
| `catenate_numbers`        | 可选      | 布尔值     | 连接数字序列。例如，`10-20-30` 会变成 `[ 102030, 10, 20, 30 ]`。默认值为 `false`。                                                                                                                                                                                                                                                                                                                                             |
| `catenate_words`          | 可选      | 布尔值     | 连接字母单词。例如，`high-speed-level` 会变成 `[ highspeedlevel, high, speed, level ]`。默认值为 `false`。                                                                                                                                                                                                                                                                                                                     |
| `generate_number_parts`   | 可选      | 布尔值     | 若为 `true`，输出中会包含纯数字词元（仅由数字组成的词元）。默认值为 `true`。                                                                                                                                                                                                                                                                                                                                                   |
| `generate_word_parts`     | 可选      | 布尔值     | 若为 `true`，输出中会包含纯字母词元（仅由字母字符组成的词元）。默认值为 `true`。                                                                                                                                                                                                                                                                                                                                               |
| `ignore_keywords`         | 可选      | 布尔值     | 是否处理标记为关键字的词元。默认值为 `false`。                                                                                                                                                                                                                                                                                                                                                                                 |
| `preserve_original`       | 可选      | 布尔值     | 在输出中，除了生成的词元外，还保留原始词元（可能包含非字母数字分隔符）。例如，`auto-drive-300` 会变成 `[ auto - drive - 300, auto, drive, 300 ]`。如果为 `true`，该过滤器会生成索引不支持的多位置词元，因此请勿在索引分词器中使用此过滤器，或者在该过滤器之后使用 `flatten_graph` 过滤器。默认值为 `false`。                                                                                                                   |
| `protected_words`         | 可选      | 字符串数组 | 指定不应被拆分的词元。                                                                                                                                                                                                                                                                                                                                                                                                         |
| `protected_words_path`    | 可选      | 字符串     | 指定一个文件的路径（绝对路径或相对于配置目录的相对路径），该文件包含不应被分隔的词元，词元需分行列出。                                                                                                                                                                                                                                                                                                                         |
| `split_on_case_change`    | 可选      | 布尔值     | 在连续字母大小写不同（一个为小写，另一个为大写）的位置拆分词元。例如，“EasySearch” 会变成 `[ Easy, Search ]`。默认值为 `true`。                                                                                                                                                                                                                                                                                                |
| `split_on_numerics`       | 可选      | 布尔值     | 在连续字母和数字的位置拆分词元。例如，`v8engine` 会变成 `[ v, 8, engine ]`。默认值为 `true`。                                                                                                                                                                                                                                                                                                                                  |
| `stem_english_possessive` | 可选      | 布尔值     | 去除英语所有格结尾，如 's。默认值为 `true`。                                                                                                                                                                                                                                                                                                                                                                                   |
| `type_table`              | 可选      | 字符串数组 | 一个自定义映射，用于指定如何处理字符以及是否将其视为分隔符，以避免不必要的拆分。例如，要将连字符（-）视为字母数字字符，可指定 `["- => ALPHA"]`，这样单词就不会在连字符处拆分。有效类型有：<br> - `ALPHA`：字母<br> - `ALPHANUM`：字母数字<br> - `DIGIT`：数字<br> - `LOWER`：小写字母<br> - `SUBWORD_DELIM`：非字母数字分隔符<br> - `UPPER`：大写字母                                                                          |
| `type_table_path`         | 可选      | 字符串     | 指定一个文件的路径（绝对路径或相对于配置目录的相对路径），该文件包含自定义字符映射。该映射指定如何处理字符以及是否将其视为分隔符，以避免不必要的拆分。有关有效类型，请参阅 `type_table`。                                                                                                                                                                                                                                      |

## 参考样例

以下示例请求创建了一个名为 `my-custom-index` 的新索引，并配置了一个带有单词分隔符图过滤器（`word_delimiter_graph`）的分词器。

```
PUT /my-custom-index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "tokenizer": "keyword",
          "filter": [ "custom_word_delimiter_filter" ]
        }
      },
      "filter": {
        "custom_word_delimiter_filter": {
          "type": "word_delimiter_graph",
          "split_on_case_change": true,
          "split_on_numerics": true,
          "stem_english_possessive": true
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
GET /my-custom-index/_analyze
{
  "analyzer": "custom_analyzer",
  "text": "FastCar's Model2023"
}
```

返回内容包含产生的词元

```
{
  "tokens": [
    {
      "token": "Fast",
      "start_offset": 0,
      "end_offset": 4,
      "type": "word",
      "position": 0
    },
    {
      "token": "Car",
      "start_offset": 4,
      "end_offset": 7,
      "type": "word",
      "position": 1
    },
    {
      "token": "Model",
      "start_offset": 10,
      "end_offset": 15,
      "type": "word",
      "position": 2
    },
    {
      "token": "2023",
      "start_offset": 15,
      "end_offset": 19,
      "type": "word",
      "position": 3
    }
  ]
}
```

## 单词分隔符图过滤器（`word_delimiter_graph`）和单词分隔符过滤器（`word_delimiter`）的区别

当以下任何参数设置为 `true` 时，单词分隔符图过滤器和单词分隔符分词过滤器都会生成跨越多个位置的词元：

- `catenate_all`
- `catenate_numbers`
- `catenate_words`
- `preserve_original`

为了说明这两种过滤器的区别，我们以输入文本 `Pro-XT500` 为例。

### 单词分隔符图过滤器（`word_delimiter_graph`）

单词分隔符图过滤器会为多位置词元分配一个 `positionLength` 属性，该属性表明一个词元跨越了多少个位置。这确保了该过滤器始终能生成有效的词元图，使其适用于高级词元图场景。虽然带有多位置词元的词元图不支持用于索引，但它们在搜索场景中仍然很有用。例如，像 `match_phrase` 这样的查询可以使用这些图从单个输入字符串生成多个子查询。对于示例输入文本，单词分隔符图过滤器会生成以下词元：

- `Pro`（位置 1）
- `XT500`（位置 2）
- `ProXT500`（位置 1，`positionLength`：2）

`positionLength` 属性使得生成的图可用于高级查询。

### 单词分隔符过滤器（`word_delimiter`）

相比之下，单词分隔符过滤器不会为多位置词元分配 `positionLength` 属性，当存在这些词元时，会导致生成无效的图。对于示例输入文本，单词分隔符过滤器会生成以下词元：

- `Pro`（位置 1）
- `XT500`（位置 2）
- `ProXT500`（位置 1，无 `positionLength`）

缺少 `positionLength` 属性会导致包含多位置词元的词元流所生成的词元图无效。

