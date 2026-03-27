---
title: "拼音分析器（Pinyin）"
date: 0001-01-01
summary: "Pinyin 分析器 #  pinyin 分析器能够在索引阶段将中文字符实时转写为拼音，并在检索阶段对拼音、汉字及混输查询进行统一匹配。借助该插件，你可以轻松实现：
相关指南（先读这些） #     文本分析基础
   文本分析：识别词元
  支持 全拼、首字母、全拼拼接 等多种检索方式；
  保留非中文字符，实现「中英混输」搜索；
  借助 token filter 在分词链中灵活组合不同策略；
  在联想输入、排序、聚合等场景下提升中文用户体验。
  适用于人名、地名、品牌、歌曲、商品等多种中文文本检索场景。
参数说明 #  下表整理了 pinyin 分词器 / 过滤器支持的全部可选参数及默认值。
   参数 说明 默认值     keep_first_letter 仅保留每个汉字的拼音首字母。例如 刘德华 → ldh true   keep_separate_first_letter 将首字母分开存储，提高命中率。例如 刘德华 → l, d, h，注意：这可能会由于词频增加而增加查询模糊性。 false   limit_first_letter_length 首字母结果最长长度 16   keep_full_pinyin 保留每个汉字的全拼。如 刘德华 → liu, de, hua true   keep_joined_full_pinyin 将全拼连接在一起。如 刘德华 → liudehua false   keep_none_chinese 保留非中文字符（数字/字母等） true   keep_none_chinese_together 将连续非中文字符作为整体保留。例如， DJ 音乐家 变成 DJ 、 yin 、 yue 、 jia 。当设置为 false 时， DJ 音乐家 变成 D 、 J 、 yin 、 yue 、 jia 。注意： keep_none_chinese 应该先启用。 true   keep_none_chinese_in_first_letter 在首字母结果中保留非中文字。例如， 刘德华 AT2016 变成 ldhat2016 。符 true   keep_none_chinese_in_joined_full_pinyin 在拼接全部拼音时保留非中文字。例如， 刘德华 2016 变成 liudehua2016 。符 false   none_chinese_pinyin_tokenize 将非中文字符中可能的拼音继续拆分例如， liudehuaalibaba13zhuanghan 变成 liu 、 de 、 hua 、 a 、 li 、 ba 、 ba 、 13 、 zhuang 、 han 。注意： keep_none_chinese 和 keep_none_chinese_together 应该先启用。 true   keep_original 同时保留原始文本 false   lowercase 对非中文字符强制小写 true   trim_whitespace 去除首尾空格 true   remove_duplicated_term 开启时，移除重复术语以节省索引空间。例如， de 的 变为 de 。注意：位置相关的查询可能会受影响。积 false   ignore_pinyin_offset 忽略 offset 以允许重叠 token。在 6."
---


# Pinyin 分析器

`pinyin` 分析器能够在索引阶段将**中文字符实时转写为拼音**，并在检索阶段对拼音、汉字及混输查询进行统一匹配。借助该插件，你可以轻松实现：

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

- 支持 **全拼**、**首字母**、**全拼拼接** 等多种检索方式；
- 保留非中文字符，实现「中英混输」搜索；
- 借助 `token filter` 在分词链中灵活组合不同策略；
- 在联想输入、排序、聚合等场景下提升中文用户体验。

适用于人名、地名、品牌、歌曲、商品等多种中文文本检索场景。

## 参数说明

下表整理了 `pinyin` 分词器 / 过滤器支持的全部可选参数及默认值。

| 参数                                      | 说明                                                                                                                                                                                                                                                                        | 默认值  |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| `keep_first_letter`                       | 仅保留每个汉字的拼音首字母。例如 `刘德华` → `ldh`                                                                                                                                                                                                                           | `true`  |
| `keep_separate_first_letter`              | 将首字母分开存储，提高命中率。例如 `刘德华` → `l`, `d`, `h`，注意：这可能会由于词频增加而增加查询模糊性。                                                                                                                                                                   | `false` |
| `limit_first_letter_length`               | 首字母结果最长长度                                                                                                                                                                                                                                                          | `16`    |
| `keep_full_pinyin`                        | 保留每个汉字的全拼。如 `刘德华` → `liu`, `de`, `hua`                                                                                                                                                                                                                        | `true`  |
| `keep_joined_full_pinyin`                 | 将全拼连接在一起。如 `刘德华` → `liudehua`                                                                                                                                                                                                                                  | `false` |
| `keep_none_chinese`                       | 保留非中文字符（数字/字母等）                                                                                                                                                                                                                                               | `true`  |
| `keep_none_chinese_together`              | 将连续非中文字符作为整体保留。例如， DJ 音乐家 变成 DJ 、 yin 、 yue 、 jia 。当设置为 false 时， DJ 音乐家 变成 D 、 J 、 yin 、 yue 、 jia 。注意： keep_none_chinese 应该先启用。                                                                                        | `true`  |
| `keep_none_chinese_in_first_letter`       | 在首字母结果中保留非中文字。例如， 刘德华 AT2016 变成 ldhat2016 。符                                                                                                                                                                                                        | `true`  |
| `keep_none_chinese_in_joined_full_pinyin` | 在拼接全部拼音时保留非中文字。例如， 刘德华 2016 变成 liudehua2016 。符                                                                                                                                                                                                     | `false` |
| `none_chinese_pinyin_tokenize`            | 将非中文字符中可能的拼音继续拆分例如， liudehuaalibaba13zhuanghan 变成 liu 、 de 、 hua 、 a 、 li 、 ba 、 ba 、 13 、 zhuang 、 han 。注意： keep_none_chinese 和 keep_none_chinese_together 应该先启用。                                                                 | `true`  |
| `keep_original`                           | 同时保留原始文本                                                                                                                                                                                                                                                            | `false` |
| `lowercase`                               | 对非中文字符强制小写                                                                                                                                                                                                                                                        | `true`  |
| `trim_whitespace`                         | 去除首尾空格                                                                                                                                                                                                                                                                | `true`  |
| `remove_duplicated_term`                  | 开启时，移除重复术语以节省索引空间。例如， de 的 变为 de 。注意：位置相关的查询可能会受影响。积                                                                                                                                                                             | `false` |
| `ignore_pinyin_offset`                    | 忽略 offset 以允许重叠 token。在 6.0 版本之后，偏移量受到严格限制，不允许重叠的标记。使用此参数，通过忽略偏移量将允许重叠的标记。请注意，所有与位置相关的查询或高亮将变得不正确。您应该使用多字段，并为不同的查询目的指定不同的设置。如果您需要偏移量，请将其设置为 false。 | `true`  |

> **备注**：以上参数可按需自由组合。若使用 `token filter`，请将其放置于自定义分词链中。

---

## 常见用例

### 创建索引并配置自定义 Pinyin Analyzer

```json
PUT /medcl
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "my_pinyin": {
          "type": "pinyin",
          "keep_separate_first_letter": false,
          "keep_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "lowercase": true,
          "remove_duplicated_term": true
        }
      },
      "analyzer": {
        "pinyin_analyzer": {
          "tokenizer": "my_pinyin"
        }
      }
    }
  }
}
```

测试分词效果

```json
GET /medcl/_analyze
{
  "text": ["刘德华"],
  "analyzer": "pinyin_analyzer"
}
```

预期返回：

```json
{
  "tokens": [
    {
      "token": "liu",
      "start_offset": 0,
      "end_offset": 1,
      "type": "word",
      "position": 0
    },
    {
      "token": "de",
      "start_offset": 1,
      "end_offset": 2,
      "type": "word",
      "position": 1
    },
    {
      "token": "hua",
      "start_offset": 2,
      "end_offset": 3,
      "type": "word",
      "position": 2
    },
    {
      "token": "刘德华",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 3
    },
    {
      "token": "ldh",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 4
    }
  ]
}
```

### 创建字段映射并建立文档

直接使用拼音分词器

```json
POST /medcl/_mapping
{
        "properties": {
            "name": {
                "type": "keyword",
                "fields": {
                    "pinyin": {
                        "type": "text",
                        "store": false,
                        "term_vector": "with_offsets",
                        "analyzer": "pinyin_analyzer",
                        "boost": 10
                    }
                }
            }
        }

}

## 索引一个文档

POST /medcl/_create/andy
{"name":"刘德华"}

## 查询测试

curl http://localhost:9200/medcl/_search?q=name:%E5%88%98%E5%BE%B7%E5%8D%8E
curl http://localhost:9200/medcl/_search?q=name.pinyin:%e5%88%98%e5%be%b7
curl http://localhost:9200/medcl/_search?q=name.pinyin:liu
curl http://localhost:9200/medcl/_search?q=name.pinyin:ldh
curl http://localhost:9200/medcl/_search?q=name.pinyin:de+hua

```

### 使用 Pinyin-TokenFilter

```
PUT /medcl1/
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "user_name_analyzer" : {
                    "tokenizer" : "whitespace",
                    "filter" : "pinyin_first_letter_and_full_pinyin_filter"
                }
            },
            "filter" : {
                "pinyin_first_letter_and_full_pinyin_filter" : {
                    "type" : "pinyin",
                    "keep_first_letter" : true,
                    "keep_full_pinyin" : false,
                    "keep_none_chinese" : true,
                    "keep_original" : false,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "trim_whitespace" : true,
                    "keep_none_chinese_in_first_letter" : true
                }
            }
        }
    }
}
```

Token 测试:刘德华 张学友 郭富城 黎明 四大天王

```
GET /medcl1/_analyze
{
  "text": ["刘德华 张学友 郭富城 黎明 四大天王"],
  "analyzer": "user_name_analyzer"
}
{
  "tokens" : [
    {
      "token" : "ldh",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "zxy",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "gfc",
      "start_offset" : 8,
      "end_offset" : 11,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "lm",
      "start_offset" : 12,
      "end_offset" : 14,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "sdtw",
      "start_offset" : 15,
      "end_offset" : 19,
      "type" : "word",
      "position" : 4
    }
  ]
}
```

### 短语查询

场景 1:全拼音查询

```
PUT /medcl2/
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "pinyin_analyzer" : {
                    "tokenizer" : "my_pinyin"
                    }
            },
            "tokenizer" : {
                "my_pinyin" : {
                    "type" : "pinyin",
                    "keep_first_letter":false,
                    "keep_separate_first_letter" : false,
                    "keep_full_pinyin" : true,
                    "keep_original" : false,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true
                }
            }
        }
    }
}

POST /medcl2/_mapping
{
  "properties": {
      "name": {
          "type": "keyword",
          "fields": {
              "pinyin": {
                  "type": "text",
                  "store": false,
                  "term_vector": "with_offsets",
                  "analyzer": "pinyin_analyzer",
                  "boost": 10
              }
          }
      }
  }
}

POST /medcl2/_doc?refresh=true
{"name":"liudehua"}


GET /medcl2/_search
{
  "query": {
    "match_phrase": {
      "name.pinyin": "刘德华"
    }
  }
}
```

场景 2:全拼/缩写/混写等多场景匹配

```
PUT /medcl3/
{
   "settings" : {
       "analysis" : {
           "analyzer" : {
               "pinyin_analyzer" : {
                   "tokenizer" : "my_pinyin"
                   }
           },
           "tokenizer" : {
               "my_pinyin" : {
                   "type" : "pinyin",
                   "keep_first_letter":true,
                   "keep_separate_first_letter" : true,
                   "keep_full_pinyin" : true,
                   "keep_original" : false,
                   "limit_first_letter_length" : 16,
                   "lowercase" : true
               }
           }
       }
   }
}

POST /medcl3/_mapping
{
  "properties": {
      "name": {
          "type": "keyword",
          "fields": {
              "pinyin": {
                  "type": "text",
                  "store": false,
                  "term_vector": "with_offsets",
                  "analyzer": "pinyin_analyzer",
                  "boost": 10
              }
          }
      }
  }
}


GET /medcl3/_analyze
{
   "text": ["刘德华"],
   "analyzer": "pinyin_analyzer"
}

POST /medcl3/_create/andy
{"name":"刘德华"}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "刘德h"
 }}
}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "刘dh"
 }}
}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "liudh"
 }}
}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "liudeh"
 }}
}

GET /medcl3/_search
{
 "query": {"match_phrase": {
   "name.pinyin": "liude华"
 }}
}

```

