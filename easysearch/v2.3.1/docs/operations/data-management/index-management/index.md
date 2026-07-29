---
title: "索引的增删改查"
date: 0001-01-01
description: "索引的创建、查看、修改、删除，以及常见的重建索引与零停机切换。"
summary: "索引的增删改查 #  索引（Index）是 Easysearch 的&quot;逻辑命名空间&quot;，但它背后是不可变段（segment）与固定主分片数等一系列硬约束。结果就是：很多你以为&quot;改一下就行&quot;的变更，最后都会走到&quot;建新索引 + 迁移数据&quot;的路上。
本页涵盖索引的完整生命周期操作：创建、查看、修改设置/映射、删除，以及重建索引的常见套路。
创建索引：把关键设置提前定好 #  你当然可以“先写一条文档让索引自动出现”，但在生产环境，更推荐显式创建索引，把关键设置、映射、分析器在写入前一次性确定：
PUT /my_index { &#34;settings&#34;: { &#34;number_of_shards&#34;: 3, &#34;number_of_replicas&#34;: 1 }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;tags&#34;: { &#34;type&#34;: &#34;keyword&#34; } } } } 你需要提前考虑的常见项：
 分片与副本：见 索引设置 映射与字段类型：见 Mapping 基础 分析器：见 Mapping 与文本分析  索引命名限制 #  Easysearch 索引命名规则：
 所有字母必须小写 索引名称不能以 _（下划线）或 -（连字符）开头 索引名称不能包含空格、逗号或以下字符：: &quot; * + / \ | ? # &gt; &lt;  是否允许“自动创建索引”？ #  对日志类场景，“按天/按月写到新索引”很常见，此时自动创建索引会很省事；但对强管控业务，自动创建索引可能变成“拼写错误导致创建一堆垃圾索引”的事故源头。"
---


# 索引的增删改查

索引（Index）是 Easysearch 的"逻辑命名空间"，但它背后是不可变段（segment）与固定主分片数等一系列硬约束。结果就是：**很多你以为"改一下就行"的变更，最后都会走到"建新索引 + 迁移数据"的路上。**

本页涵盖索引的完整生命周期操作：创建、查看、修改设置/映射、删除，以及重建索引的常见套路。

## 创建索引：把关键设置提前定好

你当然可以“先写一条文档让索引自动出现”，但在生产环境，更推荐**显式创建索引**，把关键设置、映射、分析器在写入前一次性确定：

```json
PUT /my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "tags":  { "type": "keyword" }
    }
  }
}
```

你需要提前考虑的常见项：

- **分片与副本**：见 [索引设置](./index-settings.md)
- **映射与字段类型**：见 [Mapping 基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics" >}})
- **分析器**：见 [Mapping 与文本分析]({{< relref "/docs/features/mapping-and-analysis" >}})
### 索引命名限制

Easysearch 索引命名规则：

- 所有字母必须**小写**
- 索引名称不能以 `_`（下划线）或 `-`（连字符）开头
- 索引名称不能包含空格、逗号或以下字符：`:` `"` `*` `+` `/` `\` `|` `?` `#` `>` `<`
### 是否允许“自动创建索引”？

对日志类场景，“按天/按月写到新索引”很常见，此时自动创建索引会很省事；但对强管控业务，自动创建索引可能变成“拼写错误导致创建一堆垃圾索引”的事故源头。

你可以在集群配置里关闭自动创建索引（具体配置项以你部署的 Easysearch 为准），或用更精细的白名单策略只允许某些模式的索引名自动创建。

## 删除索引：最需要“安全栅栏”的操作

删除索引是不可逆操作（除非你有快照）。基本用法：

```json
DELETE /my_index
```

也可以批量删除：

```json
DELETE /index_one,index_two
DELETE /index_*
```

### 强烈建议：要求“删除必须写全名”

为了避免误操作，一般建议开启“删除必须指定明确名称”的保护，禁止 `/_all` 和通配符这种一把梭：

- 好处：减少“手滑删库”的概率
- 代价：你需要在批量删除时显式列出目标索引名

> 具体配置项以你部署的 Easysearch 为准。生产环境建议默认开启这类破坏性操作保护。

## 查看索引信息

### 获取索引的设置、映射和别名

```json
GET /my_index
```

响应中包含索引的完整信息——设置（settings）、映射（mappings）和别名（aliases）：

```json
{
  "my_index": {
    "aliases": {},
    "mappings": {
      "properties": {
        "title": { "type": "text" },
        "tags": { "type": "keyword" }
      }
    },
    "settings": {
      "index": {
        "number_of_shards": "3",
        "number_of_replicas": "1",
        "creation_date": "..."
      }
    }
  }
}
```

查询参数：

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `flat_settings` | Boolean | `false` | 以扁平格式返回 settings |
| `include_defaults` | Boolean | `false` | 包含默认设置值 |
| `local` | Boolean | `false` | 从本地节点返回信息，不查询主节点 |
| `master_timeout` | Time | `30s` | 连接主节点的超时时间 |

### 检查索引是否存在

使用 HEAD 请求检查索引是否存在，不返回响应体：

```json
HEAD /my_index
```

- 返回 **200** 表示索引存在
- 返回 **404** 表示索引不存在

### 列出索引

使用 Cat API 以表格方式查看索引列表：

```json
GET _cat/indices?v

GET _cat/indices/logs-*?v&s=store.size:desc&h=index,health,status,pri,rep,docs.count,store.size
```

## 更新映射（Mapping）

在已有索引上添加新字段：

```json
PUT /my_index/_mapping
{
  "properties": {
    "new_field": { "type": "keyword" }
  }
}
```

> **注意**：已有字段的类型**不能修改**。如果需要更改字段类型，只能[重建索引](#什么时候必须重建索引)。

查看索引的当前映射：

```json
GET /my_index/_mapping
```

查看特定字段的映射：

```json
GET /my_index/_mapping/field/title
```

更多映射概念详见 [Mapping 基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics" >}})。

## 什么时候必须“重建索引”？

有些变更不能在原索引上“就地修补”，典型例子：

- 修改某个字段的类型（例如 `text` 改 `keyword`）
- 修改分析器/分词链（尤其是影响 token 的变更）
- 变更会导致“老数据的索引方式”与“新数据的索引方式”不一致

这类变更如果强行在原索引上继续写，会出现非常真实的“新旧数据搜索体验不一致”：同样的查询，新写入的数据命中了，老数据没命中（或反过来）。

因此正确姿势通常是：

1. 建一个新索引（新 settings/mappings/analysis）
2. 把旧索引的数据迁过去（重建索引）
3. 切流量（通常用别名做零停机切换）

## 重建索引的两种常见方式

### 方式 A：scroll + bulk（通用、可控）

思路是从旧索引分批读出文档（scroll），再用 bulk 写入新索引：

- 优点：控制力强，方便做节流、过滤、字段变换
- 缺点：需要你自己写搬运逻辑（或用现成工具）

> scroll 与 bulk 的细节分别见：[分页与排序]({{< relref "/docs/features/fulltext-search/pagination-and-sorting" >}}) 与 [Bulk API]({{< relref "/docs/features/document-operations/bulk-api" >}})。

### 方式 B：_reindex API（如果你的 Easysearch 支持）

很多部署会提供 `_reindex` 这类“服务端搬运”的能力：你给出源索引与目标索引，服务端完成复制。

- 优点：简单，少写工具代码
- 缺点：可控性相对弱；大规模数据仍要注意限速与资源占用

如果你准备在生产使用，建议先在小数据量上压测与演练，再上真实业务索引。

## 零停机切换：用别名做“换索引不换应用”

重建索引的最大痛点通常不是“搬数据”，而是“切索引名”：

- 应用写死了索引名：切换就要发版
- 多服务、多语言、多团队：协调成本爆炸

解决办法是：**应用永远只访问别名**，真实索引带版本号或时间戳。

一个典型套路：

1. 创建 `my_index_v1`，把别名 `my_index` 指向它
2. 需要变更时创建 `my_index_v2` 并迁移数据
3. 用一次原子操作把别名从 v1 切到 v2

完整示例与注意事项见：[别名（Aliases）](./aliases.md)。

## 小结

- 生产环境优先显式创建索引，把 settings/mappings/analyzers 提前确定
- 删除索引务必加“安全栅栏”，尽量避免通配符误删
- 映射/分析器等关键变更通常需要重建索引
- 重建索引常用 `scroll + bulk` 或（若支持）`_reindex`
- 零停机切换的核心是：**应用只认别名，不认真实索引名**

## 参考手册（API 与参数）

- [索引设置]({{< relref "/docs/operations/data-management/index-settings" >}})
- [索引模板]({{< relref "/docs/operations/data-management/index-templates" >}})
- [索引别名]({{< relref "/docs/operations/data-management/aliases" >}})
- [开关索引与索引限制]({{< relref "/docs/operations/data-management/open-close-index" >}})
- [克隆/缩小/拆分]({{< relref "/docs/operations/data-management/clone-shrink-split" >}})
- [Rollover]({{< relref "/docs/operations/data-management/index-rollover" >}})
- [重建数据（Reindex）]({{< relref "/docs/operations/data-management/reindex" >}})
- [备份还原（快照/恢复）]({{< relref "/docs/features/data-retention/backup-restore" >}})


