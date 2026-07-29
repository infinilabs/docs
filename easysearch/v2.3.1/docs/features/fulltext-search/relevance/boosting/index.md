---
title: "加权与调参"
date: 0001-01-01
description: "Boost、function_score 等常用手段与策略。"
summary: "加权与调参 #  当默认 _score 排序不符合业务预期时，就需要通过&quot;加权&quot;来让某些字段或条件更重要。本页给出常见手段和使用建议。
查询时权重提升 #  查询时的权重提升是可以用来影响相关度的主要工具，任意类型的查询都能接受 boost 参数。将 boost 设置为 2，并不代表最终的评分 _score 是原值的两倍；实际的权重值会经过归一化和一些其他内部优化过程。尽管如此，它确实想要表明一个提升值为 2 的句子的重要性是提升值为 1 语句的两倍。
在实际应用中，无法通过简单的公式得出某个特定查询语句的&quot;正确&quot;权重提升值，只能通过不断尝试获得。需要记住的是 boost 只是影响相关度评分的其中一个因子；它还需要与其他因子相互竞争。在前例中，title 字段相对 content 字段可能已经有一个&quot;缺省的&quot;权重提升值，这因为在字段长度归一值中，标题往往比相关内容要短，所以不要想当然的去盲目提升一些字段的权重。选择权重，检查结果，如此反复。
字段级别 Boost：标题 &gt; 标签 &gt; 正文 #  最常见的需求：
 标题匹配比正文匹配更重要 关键字段（品牌、类目）权重大于辅助字段  常见做法是在 multi_match 中给字段加权，例如：
{ &#34;query&#34;: { &#34;multi_match&#34;: { &#34;query&#34;: &#34;搜索 引擎&#34;, &#34;fields&#34;: [ &#34;title^3&#34;, &#34;tags^2&#34;, &#34;content&#34; ] } } } 或者在 bool 查询中：
GET /_search { &#34;query&#34;: { &#34;bool&#34;: { &#34;should&#34;: [ { &#34;match&#34;: { &#34;title&#34;: { &#34;query&#34;: &#34;quick brown fox&#34;, &#34;boost&#34;: 2 } } }, { &#34;match&#34;: { &#34;content&#34;: &#34;quick brown fox&#34; } } ] } } } title 查询语句的重要性是 content 查询的 2 倍，因为它的权重提升值为 2。没有设置 boost 的查询语句的值为 1。"
---


# 加权与调参

当默认 `_score` 排序不符合业务预期时，就需要通过"加权"来让某些字段或条件更重要。本页给出常见手段和使用建议。

## 查询时权重提升

查询时的权重提升是可以用来影响相关度的主要工具，任意类型的查询都能接受 `boost` 参数。将 `boost` 设置为 `2`，并不代表最终的评分 `_score` 是原值的两倍；实际的权重值会经过归一化和一些其他内部优化过程。尽管如此，它确实想要表明一个提升值为 `2` 的句子的重要性是提升值为 `1` 语句的两倍。

在实际应用中，无法通过简单的公式得出某个特定查询语句的"正确"权重提升值，只能通过不断尝试获得。需要记住的是 `boost` 只是影响相关度评分的其中一个因子；它还需要与其他因子相互竞争。在前例中，`title` 字段相对 `content` 字段可能已经有一个"缺省的"权重提升值，这因为在字段长度归一值中，标题往往比相关内容要短，所以不要想当然的去盲目提升一些字段的权重。选择权重，检查结果，如此反复。

### 字段级别 Boost：标题 > 标签 > 正文

最常见的需求：

- 标题匹配比正文匹配更重要
- 关键字段（品牌、类目）权重大于辅助字段

常见做法是在 multi_match 中给字段加权，例如：

```json
{
  "query": {
    "multi_match": {
      "query": "搜索 引擎",
      "fields": [
        "title^3",
        "tags^2",
        "content"
      ]
    }
  }
}
```

或者在 bool 查询中：

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": {
              "query": "quick brown fox",
              "boost": 2
            }
          }
        },
        {
          "match": {
            "content": "quick brown fox"
          }
        }
      ]
    }
  }
}
```

`title` 查询语句的重要性是 `content` 查询的 2 倍，因为它的权重提升值为 `2`。没有设置 `boost` 的查询语句的值为 `1`。

### 提升索引权重

当在多个索引中搜索时，可以使用参数 `indices_boost` 来提升整个索引的权重，在下面例子中，当要为最近索引的文档分配更高权重时，可以这么做：

```json
GET /docs_2014_*/_search
{
  "indices_boost": {
    "docs_2014_10": 3,
    "docs_2014_09": 2
  },
  "query": {
    "match": {
      "text": "quick brown fox"
    }
  }
}
```

这个多索引查询涵盖了所有以字符串 `docs_2014_` 开始的索引。其中，索引 `docs_2014_10` 中的所有文件的权重是 `3`，索引 `docs_2014_09` 中是 `2`，其他所有匹配的索引权重为默认值 `1`。

## 使用查询结构修改相关度

Easysearch 的查询表达式相当灵活，可以通过调整查询结构中查询语句的所处层次，从而或多或少改变其重要性。

例如，设想下面这个查询：

```
quick OR brown OR red OR fox
```

可以将所有词都放在 `bool` 查询的同一层中：

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "quick" }},
        { "term": { "text": "brown" }},
        { "term": { "text": "red"   }},
        { "term": { "text": "fox"   }}
      ]
    }
  }
}
```

这个查询可能最终给包含 `quick`、`red` 和 `brown` 的文档评分与包含 `quick`、`red`、`fox` 文档的评分相同，这里 `red` 和 `brown` 是同义词，可能只需要保留其中一个，而我们真正要表达的意思是想做以下查询：

```
quick OR (brown OR red) OR fox
```

根据标准的布尔逻辑，这与原始的查询是完全一样的，但是我们已经在组合查询中看到，`bool` 查询不关心文档匹配的程度，只关心是否能匹配。

上述查询有个更好的方式：

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "quick" }},
        { "term": { "text": "fox"   }},
        {
          "bool": {
            "should": [
              { "term": { "text": "brown" }},
              { "term": { "text": "red"   }}
            ]
          }
        }
      ]
    }
  }
}
```

现在，`red` 和 `brown` 处于相互竞争的层次，`quick`、`fox` 以及 `red OR brown` 则是处于顶层且相互竞争的词。

## boosting 查询

`boosting` 查询恰恰能解决某些场景下的问题。它仍然允许我们将某些结果包括到结果中，但是使它们降级——即降低它们原来可能应有的排名：

```json
GET /_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "text": "apple"
        }
      },
      "negative": {
        "match": {
          "text": "pie tart fruit crumble tree"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

它接受 `positive` 和 `negative` 查询。只有那些匹配 `positive` 查询的文档罗列出来，对于那些同时还匹配 `negative` 查询的文档将通过文档的原始 `_score` 与 `negative_boost` 相乘的方式降级后的结果。

为了达到效果，`negative_boost` 的值必须小于 `1.0`。在这个示例中，所有包含负向词的文档评分 `_score` 都会减半。

## constant_score 查询

有时候我们根本不关心 TF/IDF，只想知道一个词是否在某个字段中出现过。可能搜索一个度假屋并希望它能尽可能有以下设施：

- WiFi
- Garden（花园）
- Pool（游泳池）

可以用简单的 `match` 查询进行匹配：

```json
GET /_search
{
  "query": {
    "match": {
      "description": "wifi garden pool"
    }
  }
}
```

但这并不是真正的全文搜索，此种情况下，TF/IDF 并无用处。我们既不关心 `wifi` 是否为一个普通词，也不关心它在文档中出现是否频繁，关心的只是它是否曾出现过。实际上，我们希望根据房屋不同设施的数量对其排名——设施越多越好。如果设施出现，则记 `1` 分，不出现记 `0` 分。

在 `constant_score` 查询中，它可以包含查询或过滤，为任意一个匹配的文档指定评分 `1`，忽略 TF/IDF 信息：

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "constant_score": {
          "query": { "match": { "description": "wifi" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "garden" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "pool" }}
        }}
      ]
    }
  }
}
```

或许不是所有的设施都同等重要——对某些用户来说有些设施更有价值。如果最重要的设施是游泳池，那我们可以为更重要的设施增加权重：

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "constant_score": {
          "query": { "match": { "description": "wifi" }}
        }},
        { "constant_score": {
          "query": { "match": { "description": "garden" }}
        }},
        { "constant_score": {
          "boost": 2,
          "query": { "match": { "description": "pool" }}
        }}
      ]
    }
  }
}
```

`pool` 语句的权重提升值为 `2`，而其他的语句为 `1`。

> 注意：最终的评分并不是所有匹配语句的简单求和，各匹配语句的 BM25 评分会按照 `bool` 查询的规则进行组合。

## function_score 查询

`function_score` 查询是用来控制评分过程的终极武器，它允许为每个与主查询匹配的文档应用一个函数，以达到改变甚至完全替换原始查询评分 `_score` 的目的。

实际上，也能用过滤器对结果的子集应用不同的函数，这样一箭双雕：既能高效评分，又能利用过滤器缓存。

Easysearch 预定义了一些函数：

- `weight`：为每个文档应用一个简单而不被规范化的权重提升值：当 `weight` 为 `2` 时，最终结果为 `2 * _score`。
- `field_value_factor`：使用这个值来修改 `_score`，如将 `popularity` 或 `votes`（受欢迎或赞）作为考虑因素。
- `random_score`：为每个用户都使用一个不同的随机评分对结果排序，但对某一具体用户来说，看到的顺序始终是一致的。
- 衰减函数——`linear`、`exp`、`gauss`：将浮动值结合到评分 `_score` 中，例如结合 `publish_date` 获得最近发布的文档，结合 `geo_location` 获得更接近某个具体经纬度（lat/lon）地点的文档，结合 `price` 获得更接近某个特定价格的文档。
- `script_score`：如果需求超出以上范围时，用自定义脚本可以完全控制评分计算，实现所需逻辑。

如果没有 `function_score` 查询，就不能将全文查询与最新发生这种因子结合在一起评分，而不得不根据评分 `_score` 或时间 `date` 进行排序；这会相互影响抵消两种排序各自的效果。这个查询可以使两个效果融合：可以仍然根据全文相关度进行排序，但也会同时考虑最新发布文档、流行文档、或接近用户希望价格的产品。

### 按受欢迎度提升权重

设想有个网站供用户发布博客并且可以让他们为自己喜欢的博客点赞，我们希望将更受欢迎的博客放在搜索结果列表中相对较上的位置，同时全文搜索的评分仍然作为相关度的主要排序依据，可以简单的通过存储每个博客的点赞数来实现它：

```json
PUT /blogposts/_doc/1
{
  "title":   "About popularity",
  "content": "In this post we will talk about...",
  "votes":   6
}
```

在搜索时，可以将 `function_score` 查询与 `field_value_factor` 结合使用，即将点赞数与全文相关度评分结合：

```json
GET /blogposts/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes"
      }
    }
  }
}
```

`function_score` 查询将主查询和函数包括在内。主查询优先执行。`field_value_factor` 函数会被应用到每个与主 `query` 匹配的文档。每个文档的 `votes` 字段都必须有值供 `function_score` 计算。如果没有文档的 `votes` 字段有值，那么就必须使用 `missing` 属性提供的默认值来进行评分计算。

在前面示例中，每个文档的最终评分 `_score` 都做了如下修改：

```
new_score = old_score * number_of_votes
```

然而这并不会带来出人意料的好结果，全文评分 `_score` 通常处于 0 到 10 之间，有 10 个赞的博客会掩盖掉全文评分，而 0 个赞的博客的评分会被置为 0。

### modifier

一种融入受欢迎度更好方式是用 `modifier` 平滑 `votes` 的值。换句话说，我们希望最开始的一些赞更重要，但是其重要性会随着数字的增加而降低。0 个赞与 1 个赞的区别应该比 10 个赞与 11 个赞的区别大很多。

对于上述情况，典型的 `modifier` 应用是使用 `log1p` 参数值，公式如下：

```
new_score = old_score * log(1 + number_of_votes)
```

`log` 对数函数使 `votes` 赞字段的评分曲线更平滑。

带 `modifier` 参数的请求如下：

```json
GET /blogposts/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field":    "votes",
        "modifier": "log1p"
      }
    }
  }
}
```

修饰语 modifier 的值可以为：`none`（默认状态）、`log`、`log1p`、`log2p`、`ln`、`ln1p`、`ln2p`、`square`、`sqrt` 以及 `reciprocal`。

### factor

可以通过将 `votes` 字段与 `factor` 的积来调节受欢迎程度效果的高低：

```json
GET /blogposts/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field":    "votes",
        "modifier": "log1p",
        "factor":   2
      }
    }
  }
}
```

添加了 `factor` 会使公式变成这样：

```
new_score = old_score * log(1 + factor * number_of_votes)
```

`factor` 值大于 `1` 会提升效果，`factor` 值小于 `1` 会降低效果。

### boost_mode

或许将全文评分与 `field_value_factor` 函数值乘积的效果仍然可能太大，我们可以通过参数 `boost_mode` 来控制函数与查询评分 `_score` 合并后的结果，参数接受的值为：

- `multiply`：评分 `_score` 与函数值的积（默认）
- `sum`：评分 `_score` 与函数值的和
- `min`：评分 `_score` 与函数值间的较小值
- `max`：评分 `_score` 与函数值间的较大值
- `replace`：函数值替代评分 `_score`

与使用乘积的方式相比，使用评分 `_score` 与函数值求和的方式可以弱化最终效果，特别是使用一个较小 `factor` 因子时：

```json
GET /blogposts/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field":    "votes",
        "modifier": "log1p",
        "factor":   0.1
      },
      "boost_mode": "sum"
    }
  }
}
```

之前请求的公式现在变成下面这样：

```
new_score = old_score + log(1 + 0.1 * number_of_votes)
```

### max_boost

最后，可以使用 `max_boost` 参数限制一个函数的最大效果：

```json
GET /blogposts/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field":    "votes",
        "modifier": "log1p",
        "factor":   0.1
      },
      "boost_mode": "sum",
      "max_boost":  1.5
    }
  }
}
```

无论 `field_value_factor` 函数的结果如何，最终结果都不会大于 `1.5`。

> 注意：`max_boost` 只对函数的结果进行限制，不会对最终评分 `_score` 产生直接影响。

## 调参策略建议

一个相对安全的调参流程：

1. 先保证基础查询（不加权）的命中集合是合理的（能"搜到"需要的内容）
2. 再引入字段/子句级 Boost 做轻量调整
3. 需要更多业务信号时，再引入 function_score，并逐步放量

避免：

- 一次同时修改太多权重，导致难以判断是哪一项引起问题
- 把大量业务规则直接硬编码到查询里，使查询极难维护

## 小结

- 查询时的权重提升是可以用来影响相关度的主要工具
- 可以通过调整查询结构来改变查询语句的重要性
- `boosting` 查询可以降级某些结果而不是完全排除
- `constant_score` 查询忽略 TF/IDF，只关心是否匹配
- `function_score` 查询是控制评分过程的终极武器，可以结合多种因素
- 优先使用字段/子句级 Boost，保持查询结构简单清晰

下一步可以继续阅读：

- [调试与 Explain](./debug-and-explain.md)
- [相关性常用策略](./relevance-recipes.md)




