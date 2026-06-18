---
title: "快速入门"
date: 0001-01-01
description: "通过一个简单的员工目录示例，快速了解 Easysearch 的核心功能。"
summary: "快速入门 #  为了让大家对 Easysearch 能实现什么及其上手难易程度有一个基本印象，让我们从一个简单的教程开始并介绍索引、搜索及聚合等基础概念。
我们将一并介绍一些新的技术术语，即使无法立即全部理解它们也无妨，因为在本书后续内容中，我们将继续深入介绍这里提到的所有概念。
接下来尽情享受 Easysearch 探索之旅。
创建一个雇员目录 #  我们受雇于 Megacorp 公司，作为 HR 部门新的&quot;热爱无人机&quot;（&ldquo;We love our drones!&quot;）激励项目的一部分，我们的任务是为此创建一个员工目录。该目录应当能培养员工认同感及支持实时、高效、动态协作，因此有一些业务需求：
 支持包含多值标签、数值、以及全文本的数据 检索任一员工的完整信息 允许结构化搜索，比如查询 30 岁以上的员工 允许简单的全文搜索以及较复杂的短语搜索 支持在匹配文档内容中高亮显示搜索片段 支持基于数据创建和管理分析仪表盘  索引员工文档 #  第一个业务需求是存储员工数据。这将会以员工文档的形式存储：一个文档代表一个员工。存储数据到 Easysearch 的行为叫做索引，但在索引一个文档之前，需要确定将文档存储在哪里。
一个 Easysearch 集群可以包含多个索引，相应的每个索引可以包含多个文档，每个文档又有多个属性。
 注意：索引这个词在 Easysearch 语境中有多种含义：
 索引（名词）：一个索引类似于传统关系数据库中的一个数据库，是一个存储关系型文档的地方。索引的复数词为 indices 或 indexes。 索引（动词）：索引一个文档就是存储一个文档到一个索引（名词）中以便被检索和查询。这非常类似于 SQL 语句中的 INSERT 关键词，除了文档已存在时，新文档会替换旧文档情况之外。 倒排索引：Easysearch 和 Lucene 使用了一个叫做倒排索引的结构来达到快速检索的目的。默认的，一个文档中的每一个属性都是被索引的（有一个倒排索引）和可搜索的。   对于员工目录，我们将做如下操作：
 每个员工索引一个文档，文档包含该员工的所有信息 该类型位于索引 megacorp 内 该索引保存在我们的 Easysearch 集群中  实践中这非常简单（尽管看起来有很多步骤），我们可以通过一条命令完成所有这些动作："
---


# 快速入门

为了让大家对 Easysearch 能实现什么及其上手难易程度有一个基本印象，让我们从一个简单的教程开始并介绍索引、搜索及聚合等基础概念。

我们将一并介绍一些新的技术术语，即使无法立即全部理解它们也无妨，因为在本书后续内容中，我们将继续深入介绍这里提到的所有概念。

接下来尽情享受 Easysearch 探索之旅。

## 创建一个雇员目录

我们受雇于 Megacorp 公司，作为 HR 部门新的"热爱无人机"（"We love our drones!"）激励项目的一部分，我们的任务是为此创建一个员工目录。该目录应当能培养员工认同感及支持实时、高效、动态协作，因此有一些业务需求：

- 支持包含多值标签、数值、以及全文本的数据
- 检索任一员工的完整信息
- 允许结构化搜索，比如查询 30 岁以上的员工
- 允许简单的全文搜索以及较复杂的短语搜索
- 支持在匹配文档内容中高亮显示搜索片段
- 支持基于数据创建和管理分析仪表盘

## 索引员工文档

第一个业务需求是存储员工数据。这将会以员工文档的形式存储：一个文档代表一个员工。存储数据到 Easysearch 的行为叫做索引，但在索引一个文档之前，需要确定将文档存储在哪里。

一个 Easysearch 集群可以包含多个索引，相应的每个索引可以包含多个文档，每个文档又有多个属性。

> 注意：索引这个词在 Easysearch 语境中有多种含义：
> - **索引（名词）**：一个索引类似于传统关系数据库中的一个数据库，是一个存储关系型文档的地方。索引的复数词为 indices 或 indexes。
> - **索引（动词）**：索引一个文档就是存储一个文档到一个索引（名词）中以便被检索和查询。这非常类似于 SQL 语句中的 `INSERT` 关键词，除了文档已存在时，新文档会替换旧文档情况之外。
> - **倒排索引**：Easysearch 和 Lucene 使用了一个叫做倒排索引的结构来达到快速检索的目的。默认的，一个文档中的每一个属性都是被索引的（有一个倒排索引）和可搜索的。

对于员工目录，我们将做如下操作：

- 每个员工索引一个文档，文档包含该员工的所有信息
- 该类型位于索引 `megacorp` 内
- 该索引保存在我们的 Easysearch 集群中

实践中这非常简单（尽管看起来有很多步骤），我们可以通过一条命令完成所有这些动作：

```json
PUT /megacorp/_doc/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

注意，路径 `/megacorp/_doc/1` 包含了三部分的信息：

- `megacorp`：索引名称
- `_doc`：文档类型（Easysearch 中类型统一为 `_doc`）
- `1`：特定雇员的ID

请求体——JSON 文档——包含了这位员工的所有详细信息，他的名字叫 John Smith，今年 25 岁，喜欢攀岩。

很简单！无需进行执行管理任务，如创建一个索引或指定每个属性的数据类型之类的，可以直接只索引一个文档。Easysearch 默认地完成其他一切，因此所有必需的管理任务都在后台使用默认设置完成。

进行下一步前，让我们增加更多的员工信息到目录中：

```json
PUT /megacorp/_doc/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/_doc/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```

## 检索文档

目前我们已经在 Easysearch 中存储了一些数据，接下来就能专注于实现应用的业务需求了。第一个需求是可以检索到单个雇员的数据。

这在 Easysearch 中很简单。简单地执行一个 HTTP `GET` 请求并指定文档的地址——索引库和ID。使用这两个信息可以返回原始的 JSON 文档：

```json
GET /megacorp/_doc/1
```

返回结果包含了文档的一些元数据，以及 `_source` 属性，内容是 John Smith 雇员的原始 JSON 文档：

```json
{
  "_index" :   "megacorp",
  "_type" :    "_doc",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests": [ "sports", "music" ]
  }
}
```

> 提示：将 HTTP 命令由 `PUT` 改为 `GET` 可以用来检索文档，同样的，可以使用 `DELETE` 命令来删除文档，以及使用 `HEAD` 指令来检查文档是否存在。如果想更新已存在的文档，只需再次 `PUT`。

## 轻量搜索

一个 `GET` 是相当简单的，可以直接得到指定的文档。现在尝试点儿稍微高级的功能，比如一个简单的搜索！

第一个尝试的几乎是最简单的搜索了。我们使用下列请求来搜索所有雇员：

```json
GET /megacorp/_search
```

可以看到，我们仍然使用索引库 `megacorp`，但与指定一个文档 ID 不同，这次使用 `_search`。返回结果包括了所有三个文档，放在数组 `hits` 中。一个搜索默认返回十条结果。

接下来，尝试下搜索姓氏为 `Smith` 的雇员。为此，我们将使用一个查询字符串搜索，很容易通过命令行完成。这个方法一般涉及到一个查询字符串（query-string）搜索，因为我们通过一个URL参数来传递查询信息给搜索接口：

```json
GET /megacorp/_search?q=last_name:Smith
```

我们仍然在请求路径中使用 `_search` 端点，并将查询本身赋值给参数 `q=`。返回结果给出了所有的 Smith。

## 使用查询表达式搜索

Query-string 搜索通过命令非常方便地进行临时性的即席搜索，但它有自身的局限性。Easysearch 提供一个丰富灵活的查询语言叫做查询表达式（Query DSL），它支持构建更加复杂和健壮的查询。

领域特定语言（DSL），使用 JSON 构造了一个请求。我们可以像这样重写之前的查询所有名为 Smith 的搜索：

```json
GET /megacorp/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

返回结果与之前的查询一样，但还是可以看到有一些变化。其中之一是，不再使用 query-string 参数，而是一个请求体替代。这个请求使用 JSON 构造，并使用了一个 `match` 查询（属于查询类型之一，后面将继续介绍）。

## 更复杂的搜索

现在尝试下更复杂的搜索。同样搜索姓氏为 Smith 的员工，但这次我们只需要年龄大于 30 的。查询需要稍作调整，使用过滤器 `filter`，它支持高效地执行一个结构化查询。

```json
GET /megacorp/_search
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith"
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 }
                }
            }
        }
    }
}
```

这部分与我们之前使用的 `match` 查询一样。这部分是一个 `range` 过滤器，它能找到年龄大于 30 的文档，其中 `gt` 表示大于（great than）。

目前无需太多担心语法问题，后续会更详细地介绍。只需明确我们添加了一个过滤器用于执行一个范围查询，并复用之前的 `match` 查询。现在结果只返回了一名员工，叫 Jane Smith，32 岁。

## 全文搜索

截止目前的搜索相对都很简单：单个姓名，通过年龄过滤。现在尝试下稍微高级点儿的全文搜索——一项传统数据库确实很难搞定的任务。

搜索下所有喜欢攀岩（rock climbing）的员工：

```json
GET /megacorp/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

显然我们依旧使用之前的 `match` 查询在 `about` 属性上搜索 `rock climbing`。得到两个匹配的文档。

Easysearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。第一个最高得分的结果很明显：John Smith 的 `about` 属性清楚地写着 `rock climbing`。

但为什么 Jane Smith 也作为结果返回了呢？原因是她的 `about` 属性里提到了 `rock`。因为只有 `rock` 而没有 `climbing`，所以她的相关性得分低于 John 的。

这是一个很好的案例，阐明了 Easysearch 如何在全文属性上搜索并返回相关性最强的结果。Easysearch 中的相关性概念非常重要，也是完全区别于传统关系型数据库的一个概念，数据库中的一条记录要么匹配要么不匹配。

## 短语搜索

找出一个属性中的独立单词是没有问题的，但有时候想要精确匹配一系列单词或者短语。比如，我们想执行这样一个查询，仅匹配同时包含 `rock` 和 `climbing`，并且二者以短语 `rock climbing` 的形式紧挨着的雇员记录。

为此对 `match` 查询稍作调整，使用一个叫做 `match_phrase` 的查询：

```json
GET /megacorp/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

毫无悬念，返回结果仅有 John Smith 的文档。

## 高亮搜索

许多应用都倾向于在每个搜索结果中高亮部分文本片段，以便让用户知道为何该文档符合查询条件。在 Easysearch 中检索出高亮片段也很容易。

再次执行前面的查询，并增加一个新的 `highlight` 参数：

```json
GET /megacorp/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
```

当执行该查询时，返回结果与之前一样，与此同时结果中还多了一个叫做 `highlight` 的部分。这个部分包含了 `about` 属性匹配的文本片段，并以 HTML 标签 `<em></em>` 封装：

```json
{
   ...
   "hits": {
      "total": 1,
      "max_score": 0.23013961,
      "hits": [
         {
            ...
            "_score": 0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            },
            "highlight": {
               "about": [
                  "I love to go <em>rock</em> <em>climbing</em>"
               ]
            }
         }
      ]
   }
}
```

## 分析

终于到了最后一个业务需求：支持管理者对员工目录做分析。Easysearch 有一个功能叫聚合（aggregations），允许我们基于数据生成一些精细的分析结果。聚合与 SQL 中的 `GROUP BY` 类似但更强大。

举个例子，挖掘出员工中最受欢迎的兴趣爱好：

```json
GET /megacorp/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

暂时忽略掉语法，直接看看结果：

```json
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```

可以看到，两位员工对音乐感兴趣，一位对林业感兴趣，一位对运动感兴趣。这些聚合的结果数据并非预先统计，而是根据匹配当前查询的文档即时生成的。如果想知道叫 Smith 的员工中最受欢迎的兴趣爱好，可以直接构造一个组合查询：

```json
GET /megacorp/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```

`all_interests` 聚合已经变为只包含匹配查询的文档。

聚合还支持分级汇总。比如，查询特定兴趣爱好员工的平均年龄：

```json
GET /megacorp/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

得到的聚合结果有点儿复杂，但理解起来还是很简单的：

```json
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```

输出基本是第一次聚合的加强版。依然有一个兴趣及数量的列表，只不过每个兴趣都有了一个附加的 `avg_age` 属性，代表有这个兴趣爱好的所有员工的平均年龄。

即使现在不太理解这些语法也没有关系，依然很容易了解到复杂聚合及分组通过 Easysearch 特性实现得很完美，能够提取的数据类型也没有任何限制。

## 教程结语

欣喜的是，这是一个关于 Easysearch 基础描述的教程，且仅仅是浅尝辄止，更多诸如 suggestions、geolocation、percolation、fuzzy 与 partial matching 等特性均被省略，以便保持教程的简洁。但它确实突显了开始构建高级搜索功能多么容易。不需要配置——只需要添加数据并开始搜索！

很可能语法会让你在某些地方有所困惑，并且对各个方面如何微调也有一些问题。没关系！本书后续内容将针对每个问题详细解释，让你全方位地理解 Easysearch 的工作原理。

## 分布式特性

在本章开头，我们提到过 Easysearch 可以横向扩展至数百（甚至数千）的服务器节点，同时可以处理 PB 级数据。我们的教程给出了一些使用 Easysearch 的示例，但并不涉及任何内部机制。Easysearch 天生就是分布式的，并且在设计时屏蔽了分布式的复杂性。

Easysearch 在分布式方面几乎是透明的。教程中并不要求了解分布式系统、分片、集群发现或其他的各种分布式概念。可以使用笔记本上的单节点轻松地运行教程里的程序，但如果你想要在 100 个节点的集群上运行程序，一切依然顺畅。

Easysearch 尽可能地屏蔽了分布式系统的复杂性。这里列举了一些在后台自动执行的操作：

- 分配文档到不同的容器或分片中，文档可以储存在一个或多个节点中
- 按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡
- 复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失
- 将集群中任一节点的请求路由到存有相关数据的节点
- 集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复

当阅读本书时，将会遇到有关 Easysearch 分布式特性的补充章节。这些章节将介绍有关集群扩容、故障转移、应对文档存储、执行分布式搜索，以及分区（shard）及其工作原理。

这些章节并非必读，完全可以无需了解内部机制就使用 Easysearch，但是它们将从另一个角度帮助你了解更完整的 Easysearch 知识。可以根据需要跳过它们，或者想更完整地理解时再回头阅读也无妨。

## 后续步骤

现在大家对于通过 Easysearch 能够实现什么样的功能、以及上手的难易程度应该已经有了初步概念。Easysearch 力图通过最少的知识和配置做到开箱即用。学习 Easysearch 的最好方式是投入实践：尽情开始索引和搜索吧！

然而，对于 Easysearch 知道得越多，就越有生产效率。告诉 Easysearch 越多的领域知识，就越容易进行结果调优。

本书的后续内容将帮助你从新手成长为专家，每个章节不仅阐述必要的基础知识，而且包含专家建议。如果刚刚上手，这些建议可能无法立竿见影；但 Easysearch 有着合理的默认设置，在无需干预的情况下通常都能工作得很好。当你开始追求毫秒级的性能提升时，随时可以重温这些章节。

## 小结

- Easysearch 是一个分布式的实时文档存储，每个字段可以被索引与搜索
- 文档以 JSON 格式存储，支持复杂的数据结构
- 可以通过 RESTful API 与 Easysearch 交互
- 索引、搜索、聚合等操作都非常简单，开箱即用
- Easysearch 天生就是分布式的，屏蔽了分布式系统的复杂性

下一步可以继续阅读：

- [基础理论](../fundamentals/)：深入了解集群、分片、近实时等核心概念
- [写入与存储](../features/document-operations/)：学习如何高效写入数据
- [搜索](../features/fulltext-search/)：深入学习 Query DSL 和各种搜索功能
- [聚合分析](../features/aggregations/)：学习如何使用聚合进行数据分析

