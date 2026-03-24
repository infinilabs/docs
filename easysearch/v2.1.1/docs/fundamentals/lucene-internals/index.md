---
title: "Lucene 底层原理"
date: 0001-01-01
description: "理解 Easysearch 的搜索基石：Lucene 的核心概念、倒排索引、段的不可变性、文件格式与评分模型。"
summary: "Lucene 底层原理 #  Easysearch 的搜索能力建立在 Apache Lucene 之上。理解 Lucene 的核心概念，有助于更好地使用 Easysearch 和进行性能调优。
Lucene 与 Easysearch 的关系 #  用户请求 → Easysearch (分布式协调、REST API、集群管理) ↓ 每个分片 = 一个 Lucene Index ↓ Lucene (倒排索引、评分、段合并)  Easysearch 负责：分布式路由、副本复制、集群管理、REST API、安全 Lucene 负责：文本分析、倒排索引构建、查询执行、评分计算  每个 Easysearch 分片（shard） 对应一个完整的 Lucene Index。
 术语对齐：Lucene 的&quot;索引&quot;在 Easysearch 里更接近&quot;分片&quot;；而 Easysearch 的&quot;索引&quot;是多个分片的集合。
 倒排索引：全文检索的核心 #  要理解 Easysearch 的搜索行为，必须先回答一个基础问题：文本是怎么变成&quot;可搜索&quot;的？
关系型数据库通常把一个字段当作一个整体来索引；而全文检索需要把一个字段里的每个&quot;词&quot;都变成可检索的索引项。这就需要倒排索引（Inverted Index）——一个天然支持&quot;一个字段对应很多词项&quot;的数据结构。
正排 vs 倒排 #  正排索引（文档 → 词项）："
---


# Lucene 底层原理

Easysearch 的搜索能力建立在 Apache Lucene 之上。理解 Lucene 的核心概念，有助于更好地使用 Easysearch 和进行性能调优。

## Lucene 与 Easysearch 的关系

```text
用户请求 → Easysearch (分布式协调、REST API、集群管理)
               ↓
         每个分片 = 一个 Lucene Index
               ↓
         Lucene (倒排索引、评分、段合并)
```

- **Easysearch** 负责：分布式路由、副本复制、集群管理、REST API、安全
- **Lucene** 负责：文本分析、倒排索引构建、查询执行、评分计算

每个 Easysearch **分片（shard）** 对应一个完整的 Lucene Index。

> 术语对齐：Lucene 的"索引"在 Easysearch 里更接近"分片"；而 Easysearch 的"索引"是多个分片的集合。

## 倒排索引：全文检索的核心

要理解 Easysearch 的搜索行为，必须先回答一个基础问题：**文本是怎么变成"可搜索"的？**

关系型数据库通常把一个字段当作一个整体来索引；而全文检索需要把一个字段里的每个"词"都变成可检索的索引项。这就需要**倒排索引（Inverted Index）**——一个天然支持"一个字段对应很多词项"的数据结构。

### 正排 vs 倒排

**正排索引**（文档 → 词项）：

| 文档 ID | 内容 |
|---------|------|
| 1 | "搜索引擎 技术" |
| 2 | "分布式 搜索引擎" |

**倒排索引**（词项 → 文档）：

| 词项 | 文档列表 |
|------|---------|
| 搜索引擎 | [1, 2] |
| 技术 | [1] |
| 分布式 | [2] |

搜索 "搜索引擎" 时，直接查倒排索引，O(1) 定位到文档 1 和 2，无需逐条扫描。

### 倒排索引的组成

一个真实的倒排索引不仅包含"出现在哪些文档"，还会包含丰富的统计信息：

| 组件 | 作用 | 对应文件 |
|------|------|---------|
| **Term Dictionary** | 词项的有序字典 | `.tim` |
| **Term Index** | 词典的前缀索引（FST，加速查找） | `.tip` |
| **Postings List** | 每个词项的文档列表、位置、频率 | `.doc`, `.pos`, `.pay` |
| **Stored Fields** | 原始文档内容（`_source`） | `.fdt`, `.fdx` |
| **Doc Values** | 列式存储，用于排序和聚合 | `.dvd`, `.dvm` |
| **Norms** | 字段长度归一化因子（评分用） | `.nvd`, `.nvm` |

在 Easysearch 中，**每个被索引的字段，都有自己的倒排索引**——这是一个关键点。

### 示例：倒排索引匹配

假设有两个文档：

1. The quick brown fox jumped over the lazy dog
2. Quick brown foxes leap over lazy dogs in summer

构建倒排索引后（简化版）：

```text
Term      Doc_1  Doc_2
-------------------------
Quick   |       |  X
The     |   X   |
brown   |   X   |  X
dog     |   X   |
dogs    |       |  X
fox     |   X   |
foxes   |       |  X
jumped  |   X   |
lazy    |   X   |  X
leap    |       |  X
over    |   X   |  X
quick   |   X   |
summer  |       |  X
```

搜索 `quick brown`，两个文档都匹配，Doc_1 命中 2 个词条更相关。

但这个原始索引存在问题：

- `Quick` 和 `quick` 是不同的词条——用户可能认为它们相同
- `fox` 和 `foxes`、`dog` 和 `dogs` 有相同词根但被视为不同词条
- `jumped` 和 `leap` 是同义词但无法关联

这就是为什么在索引之前需要对文本进行**分析（Analysis）**：小写化、词干提取、同义词扩展等，使词条被规范为标准形式。分析后的索引会变成：

```text
Term      Doc_1  Doc_2
-------------------------
brown   |   X   |  X
dog     |   X   |  X
fox     |   X   |  X
jump    |   X   |  X
lazy    |   X   |  X
over    |   X   |  X
quick   |   X   |  X
summer  |       |  X
```

现在搜索 `Quick fox` 就能匹配到两个文档了。这个过程由分析器自动处理，详见[映射与分析入门]({{< relref "./mapping-analysis-intro.md" >}})。

## 段（Segment）与不可变性

### 不可变性：为什么倒排索引"写入后不改"

倒排索引一旦写入磁盘，就被设计为**不可变（immutable）**：不会原地修改。这带来了几个重要好处：

- **无需锁**：不存在"就地更新"，并发读写更容易
- **更友好的缓存**：文件进入操作系统缓存后可以长期有效
- **缓存更耐用**：过滤等缓存不会因为"索引被修改"而频繁失效
- **更好的压缩**：一次写入较大的不可变结构，压缩比更高、IO 更少

不可变性的代价也很直观：**你无法直接把一个新文档"插入到旧索引里"让它立刻可搜**。

### 动态更新：用"段"追加，而不是改写

Lucene 引入了"按段搜索"的概念来解决这个问题：

- 一个 **段（segment）** 本身就是一个完整的倒排索引
- 一个 Lucene 索引是多个段的集合，并通过 **提交点（commit point）** 记录"当前有哪些段是已知且可用的"

写入的概念流程：

1. 新文档先进入内存缓冲（in-memory buffer）
2. 缓冲不时被 refresh：写入一个新段到文件系统缓存
3. 新段被打开，从而使新文档"可见/可搜"
4. 定期 flush：将段 `fsync` 到磁盘，写入新的提交点

查询时会按顺序查询所有已知段，并把段级结果合并成全局结果。这样就实现了"动态更新"，而不需要重建整个大索引。

### 段的生命周期

```text
写入文档 → 内存缓冲区
           ↓ refresh (默认 1 秒)
         新段（不可变）→ 可被搜索
           ↓ flush / translog
         持久化到磁盘
           ↓ merge
         多个小段合并为大段
```

关键特性：

- **不可变性**：段一旦写入就不会被修改。更新 = 标记旧文档删除 + 写入新文档
- **近实时**：refresh 操作创建新段，新文档变为可搜索（默认 1 秒延迟）
- **段合并**：后台自动将小段合并为大段，减少段数量，回收已删除文档的空间

> 详见[写入与存储机制]({{< relref "./write-and-storage.md" >}})。

### 段数量与性能

| 段数量 | 影响 |
|--------|------|
| 太多 | 每次搜索需遍历所有段，查询变慢 |
| 太少 | 合并时消耗大量 IO 和 CPU |
| 适中 | 由 Easysearch merge policy 自动控制 |

Easysearch 默认使用 **TieredMergePolicy**（分层合并策略），其核心参数：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `floor_segment` | 2 MB | 小于此大小的段会被视为等大，鼓励合并小段 |
| `max_merge_at_once` | 10 | 单次合并操作最多合并多少个段 |
| `max_merged_segment` | 5 GB | 合并后的段最大不超过此值（太大的段不参与常规合并） |
| `segments_per_tier` | 10.0 | 每层允许的段数量，控制合并的激进程度 |
| `deletes_pct_allowed` | 33% | 段中已删除文档的比例超过此值时，会优先被选中合并 |

可通过以下 API 查看段信息：

```
GET /my-index/_segments
```

### 删除与更新：不是原地修改

段不可变，因此：

- **删除**并不会从旧段里抹掉文档，而是在 `.del`（live docs）文件里**标记删除**
- 被标记删除的文档仍可能被查询匹配到，但会在最终结果返回前被过滤掉
- **更新**也是同样语义：旧版本文档被标记删除，新版本作为新文档写入新段

## 段文件格式

每个段包含一组不可变的文件。以段名 `_0` 为例：

| 文件扩展名 | 用途 | 说明 |
|-----------|------|------|
| `.si` | Segment Info | 段的元信息（版本、文档数等） |
| `.fnm` | Field Names | 字段名称与属性 |
| `.fdx` / `.fdt` | Stored Fields | `_source` 的索引和数据 |
| `.tim` / `.tip` | Term Dictionary & Index | 词项字典与 FST 前缀索引 |
| `.doc` / `.pos` / `.pay` | Postings | 文档频率、位置、附加数据 |
| `.dvd` / `.dvm` | Doc Values | 列式存储（排序/聚合用） |
| `.nvd` / `.nvm` | Norms | 字段长度归一化因子 |
| `.liv` | Live Docs | 删除标记位图 |
| `.cfs` / `.cfe` | Compound File | 合并小文件减少文件句柄 |

索引级别的文件：

| 文件 | 用途 |
|------|------|
| `segments_N` | Commit Point，记录当前活跃的段列表 |
| `write.lock` | 写入锁，防止并发写入 |

### 倒排索引文件的查找路径

```text
搜索 "Easysearch" 的查找路径：

  .tip (FST前缀索引)
    ↓ 快速定位到 "E" 开头的块
  .tim (Term Dictionary)
    ↓ 在块内二分查找 "Easysearch"
  .doc (Postings - 频率)
    ↓ 获取包含该词的文档列表 [doc1, doc5, doc12]
  .pos (Postings - 位置)
    ↓ 获取词在每个文档中的位置（短语查询需要）
```

**FST（Finite State Transducer）**是 `.tip` 文件的核心数据结构，它是一种高度压缩的前缀树，将词项映射到 `.tim` 文件中的偏移量。FST 常驻内存，是快速查找的关键。

### 存储字段与 Doc Values

- **Stored Fields**（`.fdt` / `.fdx`）：存储文档的原始 JSON（即 `_source`），使用 LZ4 或 DEFLATE 压缩
- **Doc Values**（`.dvd` / `.dvm`）：列式存储，用于排序和聚合

```text
行式 (_source):  doc1={name:"A", age:25}  doc2={name:"B", age:30}
列式 (doc values): name: ["A", "B"]   age: [25, 30]
```

Doc Values 的关键信息：

- `keyword`、数值、日期、布尔字段默认启用 Doc Values
- `text` 字段不支持 Doc Values（使用 fielddata，不推荐）
- 不需要排序/聚合的字段可关闭以节省空间：`"doc_values": false`

### 压缩控制

```json
PUT /my-index
{
  "settings": {
    "index.codec": "best_compression"
  }
}
```

使用 DEFLATE 代替默认 LZ4，压缩率提升约 20%，但写入和读取速度会略慢。

### 磁盘空间估算

各文件类型的典型占比（仅供参考）：

| 文件类型 | 占比 | 说明 |
|----------|:----:|------|
| 存储字段 (.fdt) | 40~60% | `_source` 通常是最大的部分 |
| 倒排索引 (.tim/.doc/.pos) | 15~30% | 取决于字段数量和文本长度 |
| Doc Values (.dvd) | 5~15% | 取决于可排序/聚合字段数 |
| 其他 (norms/meta等) | 5~10% | |

## 评分模型

Lucene 使用 **BM25** 作为默认评分算法（从 Lucene 6 开始，之前是 TF-IDF）。

### BM25 核心公式

$$score(q,d) = \sum_{t \in q} IDF(t) \cdot \frac{tf \cdot (k_1 + 1)}{tf + k_1 \cdot (1 - b + b \cdot \frac{dl}{avgdl})}$$

默认参数：$k_1 = 1.2$，$b = 0.75$，`discount_overlaps = true`。

### BM25 核心因子

| 因子 | 含义 |
|------|------|
| **TF** (Term Frequency) | 词项在文档中出现的频率，越高越相关 |
| **IDF** (Inverse Document Frequency) | 词项在所有文档中的稀有程度，越稀有越重要 |
| **Field Length** | 字段越短，单个词项的权重越高 |

直觉理解：

- 一个词在文档中出现越多 → 越相关（TF）
- 一个词在整个索引中越罕见 → 越有区分度（IDF）
- 文档越短 → 单个匹配词的密度越高（Field Length）

### 查看评分解释

```json
GET /my-index/_search
{
  "explain": true,
  "query": {
    "match": { "title": "搜索引擎" }
  }
}
```

> 详见[相关性与排序]({{< relref "./relevance-and-sorting.md" >}})。

## 调优要点

| 优化点 | 说明 |
|--------|------|
| `refresh_interval` | 增大刷新间隔减少段创建频率 |
| merge policy | 调整合并策略平衡写入与查询 |
| doc values | 不需要排序/聚合的字段可关闭 `"doc_values": false` |
| `_source` | 大字段考虑 `_source.excludes` 减少存储 |
| norms | 不需要评分的字段可关闭 `"norms": false` |
| 位置信息 | 不需要短语查询时设置 `"index_options": "docs"` |
| 压缩 | 使用 `best_compression` 提升压缩率 |

## 小结

- Lucene 是 Easysearch 的底层搜索引擎，每个分片对应一个 Lucene Index
- 倒排索引把"词项 → 文档集合"作为核心结构，每个字段各自维护倒排索引
- 不可变性带来并发、缓存与压缩的优势，但意味着不能原地更新
- 动态更新依赖段（segment）：追加新段 + 更新提交点；查询合并段级结果
- FST 是 Lucene 快速查找词项的关键数据结构
- BM25 评分基于词频、逆文档频率和字段长度三个因子

下一步可以继续阅读：

- [写入与存储机制]({{< relref "./write-and-storage.md" >}})：refresh、flush、merge 详解
- [映射与分析入门]({{< relref "./mapping-analysis-intro.md" >}})：分析器与字段类型
- [相关性与排序]({{< relref "./relevance-and-sorting.md" >}})：TF/IDF、BM25 评分详解

