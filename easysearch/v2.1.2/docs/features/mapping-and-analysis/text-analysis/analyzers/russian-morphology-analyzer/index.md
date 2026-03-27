---
title: "俄语形态分析器（Russian Morphology）"
date: 0001-01-01
summary: "Russian Morphology 分析器 #  russian_morphology 分析器专为处理俄语文本而设计。与 standard 分析器不同，它能够识别俄语词汇的形态变化，将单词还原为其词干或原型（Lemmatization）。这使得用户在搜索某个单词的特定形式（如单复数、格的变化）时，能够匹配到该单词的其他形态。
该分析器由以下分词器和分词过滤器组成：
 standard 分词器：去除大部分标点符号，并依据空格和其他常见分隔符对文本进行分割。 lowercase 分词过滤器：将所有词元转换为小写，以确保匹配时不区分大小写。 russian_morphology 分词过滤器：执行俄语词汇的形态分析，将不同格、性、数的单词映射到统一原型。  相关指南（先读这些） #    文本分析：词干提取  文本分析基础  安装 #  俄语形态分词器包含在Morphological Analysis插件中。此插件已包含在Easysearch的bundle包中。
analysis-morphology插件安装命令如下：
bin/easysearch-plugin install analysis-morphology 参考样例 #  以下命令创建一个名为 my_morphology_index 的索引，并为 my_field 字段配置俄语形态分词器的索引：
PUT /my_morphology_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;my_field&#34;: { &#34;type&#34;: &#34;text&#34;, &#34;analyzer&#34;: &#34;russian_morphology&#34; } } } } 配置自定义分词器 #  在生产环境中，为了兼顾性能和准确度，建议定义一个包含小写化、形态还原和停用词过滤的自定义分析器：
PUT /my_custom_morphology_index { &#34;settings&#34;: { &#34;analysis&#34;: { &#34;analyzer&#34;: { &#34;my_analyzer&#34;: { &#34;type&#34;: &#34;custom&#34;, &#34;tokenizer&#34;: &#34;standard&#34;, &#34;filter&#34;: [ &#34;lowercase&#34;, &#34;russian_morphology&#34;, &#34;english_morphology&#34;, &#34;my_stopwords&#34; ] } }, &#34;filter&#34;: { &#34;my_stopwords&#34;: { &#34;type&#34;: &#34;stop&#34;, &#34;stopwords&#34;: &#34;_russian_&#34; } } } } } 产生的词元 #  通过形态分析，不同的词形会被索引为相同的词元。"
---


# Russian Morphology 分析器

`russian_morphology` 分析器专为处理俄语文本而设计。与 `standard` 分析器不同，它能够识别俄语词汇的形态变化，将单词还原为其词干或原型（Lemmatization）。这使得用户在搜索某个单词的特定形式（如单复数、格的变化）时，能够匹配到该单词的其他形态。

该分析器由以下分词器和分词过滤器组成：

- `standard` 分词器：去除大部分标点符号，并依据空格和其他常见分隔符对文本进行分割。
- `lowercase` 分词过滤器：将所有词元转换为小写，以确保匹配时不区分大小写。
- `russian_morphology` 分词过滤器：执行俄语词汇的形态分析，将不同格、性、数的单词映射到统一原型。

## 相关指南（先读这些）

- [文本分析：词干提取]({{< relref "/docs/features/mapping-and-analysis/text-analysis-stemming.md" >}})
- [文本分析基础]({{< relref "/docs/features/mapping-and-analysis/text-analysis/" >}})

## 安装

俄语形态分词器包含在Morphological Analysis插件中。此插件已包含在Easysearch的bundle包中。

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
        "analyzer": "russian_morphology"
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
            "russian_morphology",
            "english_morphology",
            "my_stopwords"
          ]
        }
      },
      "filter": {
        "my_stopwords": {
          "type": "stop",
          "stopwords": "_russian_"
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
  "analyzer": "russian_morphology",
  "text": "Китайские автомобили"
}
```

返回内容中包含了产生的词元：

```
{
  "tokens": [
    { "token": "китайский", "start_offset": 0, "end_offset": 9, "type": "<ALPHANUM>", "position": 0 },
    { "token": "автомобиль", "start_offset": 10, "end_offset": 20, "type": "<ALPHANUM>", "position": 1 }
  ]
}
```

### 返回结果分析
分词器会识别出“中国的（复数）”和“汽车（复数）”，并同时索引它们的原型：

|  原始单词   | 产生的词元(原型) |    说明    |
|------------|---------------|------------|
| Китайские  |   китайский   |  形容词原型  |
| автомобили |  автомобиль   | 名词单数原型 |

由于形态还原的存在，当用户搜索单数 автомобиль 时，能够精准匹配到包含复数 автомобили 的文档，这极大地提升了俄语环境下搜索的召回率。

## 更多示例

### 词义多义性

在处理某些俄语单词时，分词器会识别出该词形可能对应的多个不同词源（Lemmas）。 当一个单词在语法变形后与其他词的原型或变形发生重合时，
插件会同时索引所有可能的原型，以确保搜索的完整性。

| 原始单词 | 产生的词元(原型) | 说明                                                                    |
|--------|----------------|-------------------------------------------------------------------------|
|  Мире  |   мир, миро    | 词形重合：既可以是“世界/和平”（мир）的单数前置格，也可能是“圣油”（миро）的格位变形。|
|   из   |    из, иза     | 跨词性重合：既是常用介词“从...”（из），也可能是专有名词/人名“Иза”的单数第二格。    |

### 演示
以下脚本具体地展示了俄语形态分词器的工作原理和使用方法。

``` shell
#!/bin/sh
curl --header "Content-Type:application/json" -XDELETE 'http://localhost:9200/rustest' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest' -d '{
    "settings": {
		"analysis": {
			"analyzer": {
				"my_analyzer": {
					"type": "custom",
					"tokenizer": "standard",
					"filter": ["lowercase", "russian_morphology", "english_morphology", "my_stopwords"]
				}
			},
			"filter": {
				"my_stopwords": {
					"type": "stop",
					"stopwords": "а,без,более,бы,был,была,были,было,быть,в,вам,вас,весь,во,вот,все,всего,всех,вы,где,да,даже,для,до,его,ее,если,есть,еще,же,за,здесь,и,из,или,им,их,к,как,ко,когда,кто,ли,либо,мне,может,мы,на,надо,наш,не,него,нее,нет,ни,них,но,ну,о,об,однако,он,она,они,оно,от,очень,по,под,при,с,со,так,также,такой,там,те,тем,то,того,тоже,той,только,том,ты,у,уже,хотя,чего,чей,чем,что,чтобы,чье,чья,эта,эти,это,я,a,an,and,are,as,at,be,but,by,for,if,in,into,is,it,no,not,of,on,or,such,that,the,their,then,there,these,they,this,to,was,will,with"
				}
			}
		}
	}
}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_mapping' -d '{
	"properties" : {
		"body" : { "type" : "text", "analyzer" : "russian_morphology" },
		"text" : { "type" : "text", "analyzer" : "my_analyzer" }
	}
}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/1' -d '{"body": "У московского бизнесмена из автомобиля украли шесть миллионов рублей "}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/1' -d '{"body": "У московского бизнесмена из автомобиля украли шесть миллионов рублей "}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/2' -d '{"body": "Креативное агентство Jvision, запустило сервис, способствующий развитию автомобильного туризма в России."}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/3' -d '{"body": "Просто авто"}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/4' -d '{"body": "Китайские автомобили вновь заняли в мире первые места в рейтингах"}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/5' -d '{"body": "Январский профицит платежного баланса Китая превысил $100 млрд."}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/6' -d '{"body": "Китайская корпорация Xinmi представила новый смартфон под названием Astra Pro."}' && echo
curl --header "Content-Type:application/json" -XPOST 'http://localhost:9200/rustest/_refresh' && echo
echo "Should return 5"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "body:Китай"}}}'  | grep "_id"
echo "Should return 4, 6"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "body:Китайский"}}}'  | grep "_id"
echo "Should return 4"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "body:первый"}}}'  | grep "_id"
echo "Should return 1, 4"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "body:автомобиль"}}}'  | grep "_id"
echo "Should return 2"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "body:автомобильный"}}}'  | grep "_id"
echo "Should return 3"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "body:авто"}}}'  | grep "_id"
echo "Should return 1,2,3,4"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "body:авто*", "analyze_wildcard": true}}}'  | grep "_id"

curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/1' -d '{"text": "Curiously enough, the only thing that went through the mind of the bowl of petunias as it fell was Oh no, not again."}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/2' -d '{"text": "Many people have speculated that if we knew exactly why the bowl of petunias had thought that we would know a lot more about the nature of the Universe than we do now."}' && echo
curl --header "Content-Type:application/json" -XPUT 'http://localhost:9200/rustest/_doc/3' -d '{"text": "Не повезло только кашалоту, который внезапно возник из небытия в нескольких милях над поверхностью планеты."}' && echo
curl --header "Content-Type:application/json" -XPOST 'http://localhost:9200/rustest/_refresh' && echo
echo "Should return 3"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "text:\"миль по поверхности\"", "analyze_wildcard": true}}}'  | grep "_id"
echo "Should return 1"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "text:go", "analyze_wildcard": true}}}'  | grep "_id"
echo "Should return 2"
curl --header "Content-Type:application/json" -s 'http://localhost:9200/rustest/_search?pretty=true' -d '{"query": {"query_string": {"query": "text:thinking", "analyze_wildcard": true}}}'  | grep "_id"
echo "Searching _all field"
curl --header "Content-Type:application/json" -XPOST 'localhost:9200/rustest/_search?pretty' -d '{
  "query": {
      "query_string": {
          "query": "Китай",
          "analyze_wildcard": true
      }
  }
}'
```

