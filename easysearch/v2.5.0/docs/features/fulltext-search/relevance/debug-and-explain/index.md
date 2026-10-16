---
title: "调试与 Explain"
date: 0001-01-01
description: "Explain/Profile/Slowlog 等手段用于定位相关性问题。"
summary: "调试与 Explain #  当你觉得&quot;这条结果不该排这么前/这么后&quot;时，就需要用调试工具把 _score 拆开来看。本页给出一个通用的排查流程。
典型症状 #   明显相关的文档排在很后面 噪声文档排在前几条 修改 Mapping 或查询结构后，排序结果变得难以解释  这些问题往往源于：字段类型/分析器不匹配、查询结构不合理、boost 失衡等。
理解评分标准 #  当调试一条复杂的查询语句时，想要理解 _score 究竟是如何计算是比较困难的。Easysearch 在每个查询语句中都有一个 explain 参数，将 explain 设为 true 就可以得到更详细的信息。
GET /_search?explain { &#34;query&#34; : { &#34;match&#34; : { &#34;tweet&#34; : &#34;honeymoon&#34; }} } explain 参数可以让返回结果添加一个 _score 评分的得来依据。
 注意：增加一个 explain 参数会为每个匹配到的文档产生一大堆额外内容，但是花时间去理解它是很有意义的。如果现在看不明白也没关系——等你需要的时候再来回顾这一节就行。
 首先，我们看一下普通查询返回的元数据：
{ &#34;_index&#34; : &#34;us&#34;, &#34;_id&#34; : &#34;12&#34;, &#34;_score&#34; : 0.076713204, &#34;_source&#34; : { ... trimmed ."
---


# 调试与 Explain

当你觉得"这条结果不该排这么前/这么后"时，就需要用调试工具把 `_score` 拆开来看。本页给出一个通用的排查流程。

## 典型症状

- 明显相关的文档排在很后面
- 噪声文档排在前几条
- 修改 Mapping 或查询结构后，排序结果变得难以解释

这些问题往往源于：字段类型/分析器不匹配、查询结构不合理、boost 失衡等。

## 理解评分标准

当调试一条复杂的查询语句时，想要理解 `_score` 究竟是如何计算是比较困难的。Easysearch 在每个查询语句中都有一个 `explain` 参数，将 `explain` 设为 `true` 就可以得到更详细的信息。

```json
GET /_search?explain
{
   "query" : { "match" : { "tweet" : "honeymoon" }}
}
```

`explain` 参数可以让返回结果添加一个 `_score` 评分的得来依据。

> 注意：增加一个 `explain` 参数会为每个匹配到的文档产生一大堆额外内容，但是花时间去理解它是很有意义的。如果现在看不明白也没关系——等你需要的时候再来回顾这一节就行。

首先，我们看一下普通查询返回的元数据：

```json
{
    "_index" :      "us",
    "_id" :         "12",
    "_score" :      0.076713204,
    "_source" :     { ... trimmed ... },
    "_node" :       "mzIVYCsqSWCG_M_ZffSs9Q",
```

这里加入了该文档来自于哪个节点哪个分片上的信息，这对我们是比较有帮助的，因为词频率和文档频率是在每个分片中计算出来的，而不是每个索引中。

然后它提供了 `_explanation`。每个入口都包含一个 `description`、`value`、`details` 字段，它分别告诉你计算的类型、计算结果和任何我们需要的计算细节。

```json
"_explanation": {
   "description": "weight(tweet:honeymoon in 0) [PerFieldSimilarity], result of:",
   "value":       0.076713204,
   "details": [
      {
         "description": "fieldWeight in 0, product of:",
         "value":       0.076713204,
         "details": [
            {
               "description": "tf(freq=1.0), with freq of:",
               "value":       1,
               "details": [
                  {
                     "description": "termFreq=1.0",
                     "value":       1
                  }
               ]
            },
            {
               "description": "idf(docFreq=1, maxDocs=1)",
               "value":       0.30685282
            },
            {
               "description": "fieldNorm(doc=0)",
               "value":        0.25
            }
         ]
      }
   ]
}
```

- `honeymoon` 相关性评分计算的总结
- 检索词频率：检索词 `honeymoon` 在这个文档的 `tweet` 字段中的出现次数
- 反向文档频率：检索词 `honeymoon` 在索引上所有文档的 `tweet` 字段中出现的次数
- 字段长度准则：在这个文档中，`tweet` 字段内容的长度——内容越长，值越小

复杂的查询语句解释也非常复杂，但是包含的内容与上面例子大致相同。通过这段信息我们可以了解搜索结果是如何产生的。

> 提示：JSON 形式的 `explain` 描述是难以阅读的，但是转成 YAML 会好很多，只需要在参数中加上 `format=yaml`。

> 警告：输出 `explain` 结果代价是十分昂贵的，它只能用作调试工具。千万不要用于生产环境。

## 理解文档是如何被匹配到的

当 `explain` 选项加到某一文档上时，`explain` API 会帮助你理解为何这个文档会被匹配，更重要的是，一个文档为何没有被匹配。

请求路径为 `/index/_doc/id/_explain`，如下所示：

```json
GET /us/_doc/12/_explain
{
   "query" : {
      "bool" : {
         "filter" : { "term" :  { "user_id" : 2           }},
         "must" :  { "match" : { "tweet" :   "honeymoon" }}
      }
   }
}
```

这个 API 会返回一个 `explanation`，说明为什么这个文档匹配或不匹配。如果文档不匹配，`explanation` 会说明原因。

## 理解查询语句

对于合法查询，使用 `explain` 参数将返回可读的描述，这对准确理解 Easysearch 是如何解析你的 query 是非常有用的：

```json
GET /_validate/query?explain
{
   "query": {
      "match" : {
         "tweet" : "really powerful"
      }
   }
}
```

我们查询的每一个 index 都会返回对应的 `explanation`，因为每一个 index 都有自己的映射和分析器：

```json
{
  "valid" :         true,
  "_shards" :       { ... },
  "explanations" : [ {
    "index" :       "us",
    "valid" :       true,
    "explanation" : "tweet:really tweet:powerful"
  }, {
    "index" :       "gb",
    "valid" :       true,
    "explanation" : "tweet:realli tweet:power"
  } ]
}
```

从 `explanation` 中可以看出，匹配 `really powerful` 的 `match` 查询被重写为两个针对 `tweet` 字段的 single-term 查询，一个 single-term 查询对应查询字符串分出来的一个 term。

当然，对于索引 `us`，这两个 term 分别是 `really` 和 `powerful`，而对于索引 `gb`，term 则分别是 `realli` 和 `power`。之所以出现这个情况，是由于我们将索引 `gb` 中 `tweet` 字段的分析器修改为 `english` 分析器。

## 用 Explain 看清得分构成

Explain 能展示某个文档的 `_score` 由哪些部分组成，例如：

- 哪些子查询命中了
- 每个子查询分别贡献了多少分
- 字段长度、词频、逆文档频率等因素的影响

排查思路：

1. 选出"排序异常"的代表文档（一个排得太低，一个排得太高）
2. 对它们运行 explain，查看每个命中子句的得分构成
3. 对比两个文档之间的差异：是某个字段权重过高？某个词的影响被放大或忽略？

这能帮助你回答：**是评分机制本身的问题，还是 Mapping/查询结构的问题？**

## 用 Profile 看执行结构与耗时

Profile 工具偏向"执行计划与性能"：它会告诉你：

- 每个子查询执行了多久
- 哪些部分最耗时
```json
GET /my_index/_search
{
  "profile": true,
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "搜索引擎" } }
      ],
      "filter": [
        { "term": { "status": "published" } }
      ]
    }
  }
}
```

返回结果中的 `profile` 字段将包含每个分片上各查询组件的耗时明细（单位：纳秒），帮助你定位性能瓶颈。
在相关性调试中，它可以帮助你：

- 发现那些几乎不贡献结果却非常耗时的子句
- 确认是否有不必要的嵌套、重复计算

通常流程是：先用慢查询日志找到"慢"，再用 Profile 找出"为什么慢"，然后再结合 Explain 调整结构。

## 慢查询日志（Slowlog）

慢查询日志是发现问题的入口之一：

- 记录超过阈值的查询请求
- 帮你锁定最值得优化的那一小部分查询

通过索引设置配置慢查询日志阈值：

```json
PUT /my_index/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s",
  "index.search.slowlog.threshold.query.debug": "2s",
  "index.search.slowlog.threshold.query.trace": "500ms",
  "index.search.slowlog.threshold.fetch.warn": "1s",
  "index.search.slowlog.threshold.fetch.info": "800ms",
  "index.search.slowlog.threshold.fetch.debug": "500ms",
  "index.search.slowlog.threshold.fetch.trace": "200ms"
}
```

> 慢查询日志会记录到 Easysearch 的日志文件中，可通过日志配置文件控制输出位置和格式。

建议：

- 为重要索引配置合理的慢查询阈值（读写分别设置）
- 定期回顾慢查询日志，筛选出"又慢又频繁"的查询模式，优先优化

## 推荐调试流程

一个可重复使用的调试流程：

1. **发现问题**：通过用户反馈、监控或慢查询日志，锁定有问题的查询和索引
2. **构造样本**：收集少量具有代表性的请求和预期结果
3. **Explain**：对正常/异常结果分别执行 explain，比对 `_score` 构成
4. **Profile**：在性能相关问题时，用 profile 分析执行时间和结构
5. **调整**：根据分析结果修改：
   - Mapping（字段类型、分析器、multi-fields 设计）
   - 查询结构（must/should/filter、子句拆分/合并）
   - Boost 策略（字段/子句权重、function_score 等）
6. **验证与回归**：在测试环境或小流量下验证改动效果，避免引入新的问题

通过这样的过程，可以把"感觉不对"的相关性问题，转化为可解释、可复现、可回滚的技术改动。

## 调试相关度是最后 10% 要做的事情

理解评分过程是非常重要的，这样就可以根据具体的业务对评分结果进行调试、调节、减弱和定制。

实践中，简单的查询组合就能提供很好的搜索结果，但是为了获得具有成效的搜索结果，就必须反复推敲修改前面介绍的这些调试方法。

通常，经过对策略字段应用权重提升，或通过对查询语句结构的调整来强调某个句子的重要性这些方法，就足以获得良好的结果。有时，如果 Easysearch 基于词的 TF/IDF 模型不再满足评分需求（例如希望基于时间或距离来评分），则需要更具侵略性的调整。

除此之外，相关度的调试就有如兔子洞，一旦跳进去就很难再出来。最相关这个概念是一个难以触及的模糊目标，通常不同人对文档排序又有着不同的想法，这很容易使人陷入持续反复调整而没有明显进展的怪圈。

我们强烈建议不要陷入这种怪圈，而要监控测量搜索结果。监控用户点击最顶端结果的频次，这可以是前 10 个文档，也可以是第一页的；用户不查看首次搜索的结果而直接执行第二次查询的频次；用户来回点击并查看搜索结果的频次，等等诸如此类的信息。

这些都是用来评价搜索结果与用户之间相关程度的指标。如果查询能返回高相关的文档，用户会选择前五中的一个，得到想要的结果，然后离开。不相关的结果会让用户来回点击并尝试新的搜索条件。

一旦有了这些监控手段，想要调试查询就并不复杂，稍作调整，监控用户的行为改变并做适当反复尝试。本章介绍的一些工具就只是工具而已，要想物尽其用并将搜索结果提高到极高的水平，唯一途径就是需要具备能评价度量用户行为的强大能力。

## 小结

- `explain` API 可以帮助理解文档的评分构成和匹配原因
- `validate` API 可以帮助理解查询是如何被解析的
- Profile 工具可以帮助分析查询的性能问题
- 慢查询日志是发现问题的入口
- 监控用户行为是评价搜索结果质量的关键

下一步可以继续阅读：

- [相关性常用策略](./relevance-recipes.md)
- [评分基础](./scoring-basics.md)
- [加权与调参](./boosting.md)




