---
title: "IK 中文分析器（IK）"
date: 0001-01-01
summary: "IK 分析器 #  IK 分析器是一款专为处理中文文本设计的分析器，高效且智能。支持 ik_smart 和 ik_max_word 两种分词模式。
相关指南（先读这些） #    文本分析基础  文本分析：识别词元  IK 分词器安装 #  IK 分词插件安装命令如下：
bin/easysearch-plugin install analysis-ik 同时也需要安装 ingest-common 插件：
bin/easysearch-plugin install ingest-common 如果觉得比较麻烦，也可以直接使用 Easysearch 的 bundle 包安装部署。
使用样例 #  下面的命令样例展示了 IK 的使用方式。
# 1.创建索引 PUT index_ik # 2.创建映射关系 POST index_ik/_mapping { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;ik_max_word&#34;, &#34;search_analyzer&#34;: &#34;ik_smart&#34; } } } # 3.写入文档 POST index_ik/_create/1 {&#34;content&#34;:&#34;美国留给伊拉克的是个烂摊子吗&#34;} POST index_ik/_create/2 {&#34;content&#34;:&#34;公安部：各地校车将享最高路权&#34;} POST index_ik/_create/3 {&#34;content&#34;:&#34;中韩渔警冲突调查：韩警平均每天扣1艘中国渔船&#34;} POST index_ik/_create/4 {&#34;content&#34;:&#34;中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首&#34;} # 4."
---


# IK 分析器

IK 分析器是一款专为处理中文文本设计的分析器，高效且智能。支持 `ik_smart` 和 `ik_max_word` 两种分词模式。

## 相关指南（先读这些）

- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})
- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})

## IK 分词器安装

IK 分词插件安装命令如下：

```
bin/easysearch-plugin install analysis-ik
```

同时也需要安装 ingest-common 插件：

```
bin/easysearch-plugin install ingest-common
```

如果觉得比较麻烦，也可以直接使用 Easysearch 的 bundle 包安装部署。

## 使用样例

下面的命令样例展示了 IK 的使用方式。

```
# 1.创建索引

PUT index_ik

# 2.创建映射关系

POST index_ik/_mapping
{
    "properties": {
        "content": {
            "type": "text",
            "analyzer": "ik_max_word",
            "search_analyzer": "ik_smart"
        }
    }
}
# 3.写入文档

POST index_ik/_create/1
{"content":"美国留给伊拉克的是个烂摊子吗"}

POST index_ik/_create/2
{"content":"公安部：各地校车将享最高路权"}

POST index_ik/_create/3
{"content":"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}

POST index_ik/_create/4
{"content":"中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"}

# 4.高亮查询

POST index_ik/_search
{
    "query" : { "match" : { "content" : "中国" }},
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "content" : {}
        }
    }
}

# 返回内容

{
    "took": 14,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 2,
        "hits": [
            {
                "_index": "index",
                "_type": "fulltext",
                "_id": "4",
                "_score": 2,
                "_source": {
                    "content": "中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"
                },
                "highlight": {
                    "content": [
                        "<tag1>中国</tag1>驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首 "
                    ]
                }
            },
            {
                "_index": "index",
                "_type": "fulltext",
                "_id": "3",
                "_score": 2,
                "_source": {
                    "content": "中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"
                },
                "highlight": {
                    "content": [
                        "均每天扣1艘<tag1>中国</tag1>渔船 "
                    ]
                }
            }
        ]
    }
}
```

## 关于 ik_smart 和 ik_max_word

`ik_smart` 和 `ik_max_word` 是 IK 分词器的两个分词模式。

ik_max_word：将文本根据词典内容做最细粒度的切分。例如,ik_max_word 会将”中华人民共和国国歌”切分到“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国、共和,和,国国,国歌”,详尽生成各种可能的组合。

ik_smart：执行文本粗粒度的切分。例如：ik_smart 会把 ”中华人民共和国国歌”到“中华人民共和国,国歌”。

ik_smart 比较适合短语查询，而 ik_max_word 更合适精准匹配。值得注意的是，由于 ik_smart 做了分词切割的优化，其的分词结果并不是 ik_max_word 的分词结果的子集。

## 字段级别词典设置

Easysearch 在 IK 分词器原有的功能上增加了自定义字段级别词典的功能。

自定义字段级别词典的功能支持用户对不同的字段设置不同的分词词库，用户既可以指定 IK 完全使用自有的词库，也支持在 IK 默认的词库上增加自定义的词库内容。

### 自定义词库内容

默认的词库索引是 .analysis_ik 索引，IK 插件自动初始化的 .analysis_ik 索引。

用户可以自定义使用某个索引替代 .analysis_ik（设置参数下面会提及），但是要保持和 .analysis_ik 一个的 mapping 结构和使用预设的 pipeline：ik_dicts_default_date_pipeline。

.analysis_ik 词库需要存储的格式如下：

```json
POST .analysis_ik/_doc
{
  "dict_key": "test_dic",
  "dict_type": "main_dicts",
  "dict_content":   """中华人民共和国
中文万岁
秋水共长天"""
}
```

主要使用字段

- dict_content：词典内容字段。各个词典以换行符分隔。
- dict_key：自定义词典名。对应自定义词典中设置的 dict_key。
- dict_type：字典类型，可选 "main_dicts", "stopwords_dicts", "quantifier_dicts" 三个值。其中任意 dict_key 的"main_dicts"必须存在。

如果是对默认词库的自定义增加，dict_key 就写"default"，同时需要使用默认的词库索引 .analysis_ik 。

### 设置自定义词库

自定义词库的生效主要通过自定义 tokenizer 进行设置。

```json
PUT my-index-000001
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "ik_max_word",
          "custom_dict_enable": true,
          "load_default_dicts":true,
          "lowcase_enable": true,
          "dict_key": "test_dic",
          "dict_index":"custom_index"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "test_ik": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}
```

其中

- custom_dict_enable：布尔值，默认 false，true 则可以定制词典读取路径，否则 load_default_dicts / dict_key / dict_index 均失效。
- load_default_dicts：布尔值，默认 true，定制的词典是否包含默认的词典库。
- lowcase_enable：布尔值，默认为 true，是否大小写敏感，false 则保留原来文本的大小写。
- dict_key：string。对应词库索引中的 dict_key 字段内容。**如果词典名不匹配，则会装载错误或者直接报错** 。
- dict_index: string。词库索引名称，默认是 .analysis_ik。可以自定义，但是要保持和 mapping 结构以及 pipeline 一致。

### 词库内容的自动新增

词库的追加内容是能自动被程序探测的，这个主要依赖于 .analysis_ik 的时间戳字段和 pipeline 执行。

ik 只对词库索引的新增词典内容进行自动追加，不会对存量词典的修改进修改或者删除。如果需要对某个存量自定义词典进行修改或者删除，请进行全量 reload。

```json
# 词典索引写入需要的默认时间戳 pipeline
GET _ingest/pipeline/ik_dicts_default_date_pipeline
{
  "ik_dicts_default_date_pipeline": {
    "processors": [
      {
        "set": {
          "field": "upload_dicts_timestamp",
          "value": "{{_ingest.timestamp}}",
          "override": true
        }
      }
    ]
  }
}

# 词典索引的结构
GET .analysis.ik
{
  ".analysis.ik": {
    "aliases": {},
    "mappings": {
      "properties": {
        "dict_content": {
          "type": "text",
          "analyzer": "custom_analyzer"
        },
        "dict_key": {
          "type": "keyword"
        },
        "dict_type": {
          "type": "keyword"
        },
        "upload_dicts_timestamp": {
          "type": "date"
        }
      }
    },
    "settings": {
      "index": {
        "number_of_shards": "1",
        "provided_name": ".analysis.ik",
        "default_pipeline": "ik_dicts_default_date_pipeline",
        "analysis": {
          "analyzer": {
            "custom_analyzer": {
              "type": "custom",
              "tokenizer": "pattern_tokenizer"
            }
          },
          "tokenizer": {
            "pattern_tokenizer": {
              "pattern": "\n",
              "type": "pattern"
            }
          }
        },
        "number_of_replicas": "1"
      }
    }
  }
}
```

这里 ik_dicts_default_date_pipeline 会对每一条写入词库的数据赋予当前 upload_dicts_timestamp 时间戳。ik 会记录当前词库的最大时间戳，然后**每分钟**都会去查询一次词库索引现有的最大时间戳。如果查到词库索引的最大的时间戳大于上次记录到的时间戳，则对这段时间内的词库内容都进行加载。

### 全量 reload

reload API 通过对词典库的全量重新加载来实现词典库的更新或者删除。用户可以通过下面的命令实现：

```
# 测试索引准备

PUT my-index-000001
{
  "settings": {
    "number_of_shards": 3,
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {

          "type": "ik_smart",
          "custom_dict_enable": true,
          "load_default_dicts":false, # 这里不包含默认词库
          "lowcase_enable": true,
          "dict_key": "test_dic"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "test_ik": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}

# 原来词库分词效果，只预置了分词“自强不息”
GET my-index-000001/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text":"自强不息，杨树林"
}

{
  "tokens": [
    {
      "token": "自强不息",
      "start_offset": 0,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "杨",
      "start_offset": 5,
      "end_offset": 6,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "树",
      "start_offset": 6,
      "end_offset": 7,
      "type": "CN_CHAR",
      "position": 2
    },
    {
      "token": "林",
      "start_offset": 7,
      "end_offset": 8,
      "type": "CN_CHAR",
      "position": 3
    }
  ]
}

# 更新词库
POST .analysis_ik/_doc
{
  "dict_key": "test_dic",
  "dict_type": "main_dicts",
  "dict_content":"杨树林"
}
# 删除词库，词库文档的id为coayoJcBFHNnLYAKfTML
DELETE .analysis_ik/_doc/coayoJcBFHNnLYAKfTML?refresh=true

# 重载词库
POST _ik/_reload
{}

# 更新后的词库效果
GET my-index-000001/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text":"自强不息，杨树林"
}

{
  "tokens": [
    {
      "token": "自",
      "start_offset": 0,
      "end_offset": 1,
      "type": "CN_CHAR",
      "position": 0
    },
    {
      "token": "强",
      "start_offset": 1,
      "end_offset": 2,
      "type": "CN_CHAR",
      "position": 1
    },
    {
      "token": "不",
      "start_offset": 2,
      "end_offset": 3,
      "type": "CN_CHAR",
      "position": 2
    },
    {
      "token": "息",
      "start_offset": 3,
      "end_offset": 4,
      "type": "CN_CHAR",
      "position": 3
    },
    {
      "token": "杨树林",
      "start_offset": 5,
      "end_offset": 8,
      "type": "CN_WORD",
      "position": 4
    }
  ]
}
```

这里是实现索引里全部的词库更新。

也可以实现某个单独的词库全量更新

```
POST _ik/_reload
{"dict_key":"test_dic”}
```

