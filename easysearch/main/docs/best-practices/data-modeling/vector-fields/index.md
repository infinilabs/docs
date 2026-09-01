---
title: "向量字段建模"
date: 0001-01-01
description: "Easysearch 2.4.0 原生 HNSW 向量字段的维度、字段数量、写入、迁移和容量设计。"
summary: "向量字段建模 #  本页说明如何为 Easysearch 2.4.0 原生 HNSW 设计向量字段。新索引使用 dense_vector 和 knn 查询，无需安装 k-NN 插件。 已有旧插件索引仍使用 knn_dense_float_vector、knn_sparse_bool_vector 和 knn_nearest_neighbors，两套字段与查询语法不能混用。
完整参数和查询合同分别见 dense_vector 字段类型和 原生 HNSW 搜索。
基本映射 #  典型语义搜索文档同时保存全文字段、过滤字段和一个向量字段：
PUT /documents-v1 { &#34;settings&#34;: { &#34;number_of_shards&#34;: 2, &#34;number_of_replicas&#34;: 1 }, &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;content&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;category&#34;: { &#34;type&#34;: &#34;keyword&#34; }, &#34;embedding&#34;: { &#34;type&#34;: &#34;dense_vector&#34;, &#34;dims&#34;: 384, &#34;index&#34;: true, &#34;similarity&#34;: &#34;cosine&#34; } } } }  重要：上述 mapping 省略了 index_options，但向量仍会使用 HNSW 索引。 Easysearch 2."
---


# 向量字段建模

本页说明如何为 Easysearch 2.4.0 原生 HNSW 设计向量字段。新索引使用 `dense_vector` 和 `knn` 查询，无需安装 k-NN 插件。
已有旧插件索引仍使用 `knn_dense_float_vector`、`knn_sparse_bool_vector` 和 `knn_nearest_neighbors`，两套字段与查询语法不能混用。

完整参数和查询合同分别见 [dense_vector 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/dense-vector.md" >}})和
[原生 HNSW 搜索]({{< relref "/docs/features/vector-search/native-hnsw.md" >}})。

## 基本映射

典型语义搜索文档同时保存全文字段、过滤字段和一个向量字段：

```json
PUT /documents-v1
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "content": {
        "type": "text"
      },
      "category": {
        "type": "keyword"
      },
      "embedding": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,
        "similarity": "cosine"
      }
    }
  }
}
```

> **重要：上述 mapping 省略了 `index_options`，但向量仍会使用 HNSW 索引。** Easysearch 2.4.0 默认使用并回显
> `hnsw(m=16, ef_construction=100)`。只有需要调整构图参数时，才需要显式设置 `index_options`。

Easysearch 2.4.0 的原生 `dense_vector` 有以下建模约束：

| 决策 | 2.4.0 合同 |
| --- | --- |
| 维度 | `dims` 必填，范围为 1–4096，写入和查询向量必须完全一致 |
| 元素类型 | 仅支持 `float` |
| 索引 | 必须显式设置 `index: true` |
| 索引类型 | `index_options` 可省略；默认使用 `hnsw(m=16, ef_construction=100)` |
| 字段值 | 单值向量；同一字段不能保存多个向量 |
| 相似度 | `cosine`、`dot_product`、`l2_norm` 或 `max_inner_product` |
| 存储 | 向量保留在 `_source`，同时写入 Lucene 向量索引；不支持 doc values、排序或聚合 |

## 选择维度和相似度

`dims` 由 Embedding 模型输出决定，不能在 Easysearch 中任意缩放。更换维度意味着更换字段或新建索引，不能原地修改已有字段。

仅计算 float32 数据时，每条向量至少需要约 `dims × 4` 字节。这个数值不包含 `_source` 中的 JSON、HNSW 图、Lucene 元数据、
其他字段、segment 合并临时空间和副本，不能直接当作最终 store size。

| 维度 | float32 原始数据下限/文档 |
| ---: | ---: |
| 128 | 512 B |
| 384 | 1.5 KiB |
| 768 | 3 KiB |
| 1536 | 6 KiB |

选择模型时先以业务 Recall、NDCG 等效果指标为门槛，再比较写入吞吐、查询延迟和存储。不要仅根据“常见维度”决定生产模型，也不要
假定更高维度一定带来更好的业务效果。

相似度必须与模型训练和输出约定一致：

- `cosine` 适合按夹角比较的 Embedding；零向量会被拒绝，Easysearch 会为索引和查询执行归一化；
- `dot_product` 要求写入和查询向量均为单位向量；
- `l2_norm` 使用欧氏距离，适合未归一化的连续向量；
- `max_inner_product` 允许向量幅值影响排序。

## 单向量和多向量

只有线上需要独立查询的语义视角才应建立独立向量字段。例如标题和正文确实需要分别召回时，可以使用两个字段：

```json
{
  "mappings": {
    "properties": {
      "title_embedding": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,
        "similarity": "cosine",
        "index_options": {
          "type": "hnsw",
          "m": 16,
          "ef_construction": 100
        }
      },
      "content_embedding": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,
        "similarity": "cosine",
        "index_options": {
          "type": "hnsw",
          "m": 16,
          "ef_construction": 100
        }
      }
    }
  }
}
```

每个 `dense_vector` 字段都会为每个 segment 建立独立的 HNSW 图，增加写入 CPU、store、文件系统缓存需求和查询成本。模型 A/B
测试可以暂时双写两个字段，但验证完成后应通过新索引移除不再使用的字段。

Easysearch 2.4.0 不支持 `dense_vector` 的 `index: false`。如果只想在 `_source` 中保留一份不参与 HNSW 查询的浮点数组，可使用
普通 `float` 字段并关闭索引和 doc values：

```json
"embedding_archive": {
  "type": "float",
  "index": false,
  "doc_values": false
}
```

这种普通浮点数组不能用于原生 `knn` 查询。

## 写入和一致性

向量通常由外部 Embedding 服务生成。无论同步生成还是异步回填，都应在写入前验证：

- 模型版本与目标字段一致；
- 数组长度等于 `dims`；
- 所有元素都是有限数值；
- `dot_product` 向量已经归一化，`cosine` 向量不是零向量；
- 文档中的文本版本和向量版本可以追踪，避免文本已更新而向量仍然陈旧。

初始导入和大规模回填应使用 Bulk，并检查每个 Bulk item 的状态。HTTP 请求成功不代表所有文档都已写入。可以根据业务需要延长
`refresh_interval`、暂时减少副本，但导入结束后必须恢复生产设置，等待集群健康，并用代表性查询验证 Recall 和错误率。

异步生成时，文档可以先不包含向量字段；向量生成完成后再通过 Update 写入。建议增加模型版本和处理状态字段。
以下仅展示文档结构，向量元素已省略，不能直接作为写入请求发送；实际写入必须提供完整的 384 维向量：

```jsonc
{
  "embedding_model": "text-model-v3",
  "embedding_status": "ready",
  "embedding": [0.12, -0.03, 0.08 /* 省略其余元素，共 384 维 */]
}
```

应用查询时应过滤或跳过尚未生成向量的文档，并监控待处理数量和最长等待时间。

## 模型升级和迁移

以下变化需要新字段或新索引，不能直接修改已有 `dense_vector` 字段：

- Embedding 维度或相似度变化；
- 从旧 k-NN 插件字段迁移到原生 HNSW；
- 希望减小 HNSW 的 `m`；
- 希望移除旧向量字段或回收其 store。

短期模型 A/B 可以在同一新索引中增加 `embedding_v2` 并双写；要彻底回收旧字段空间，仍需创建目标索引并 Reindex。迁移时保留模型
版本，核对文档数、失败项、Recall、延迟和 store，再通过 alias 原子切换。完整步骤见
[迁移现有索引]({{< relref "/docs/features/vector-search/native-hnsw.md#迁移现有索引" >}})。

## 查询建模

需要与全文查询组合时，应先确定融合语义：

- 需要全文召回与全局 kNN top-k 取 OR 并集、重叠文档分数相加时，并列使用一个普通顶层 `query` 和一个顶层 `knn`；
- 需要 AND、`minimum_should_match` 或更复杂的布尔关系时，将 query-level `knn` 放入 `bool` 等复合查询。

只约束向量候选的过滤条件应放在 `knn.filter` 中，以便在 HNSW 搜索期间预过滤。如果同一业务约束必须同时作用于全文和向量
分支，应在两个分支中分别表达。下面是使用 query-level `knn` 的复合查询结构示例。其中向量仅展示前三个元素，
不能直接执行；实际查询必须提供完整的 384 维向量：

```jsonc
POST /documents-v1/_search
{
  "size": 10,
  "query": {
    "bool": {
      "must": {
        "knn": {
          "field": "embedding",
          "query_vector": [0.12, -0.03, 0.08 /* 省略其余元素，共 384 维 */],
          "k": 10,
          "num_candidates": 100,
          "filter": {
            "term": {
              "category": "manual"
            }
          }
        }
      },
      "should": {
        "match": {
          "content": {
            "query": "向量索引配置",
            "boost": 0.5
          }
        }
      }
    }
  }
}
```

该示例要求 query-level `knn` 命中，全文 `match` 只参与加分。如果要求向量和全文都命中，应将两个子句都放入 `bool.must`。

`num_candidates` 通常越大 Recall 越高，但 CPU 和延迟也越高。使用代表真实业务的查询集调参，并同时记录 Recall、P50/P95/P99、
QPS 和错误率。顶层 `knn` 适合在多分片候选中协调选取全局 top-k。Easysearch 2.4.0 也支持一个普通顶层
`query` 与一个顶层 `knn` 并列使用：两个分支取 OR 并集，重叠文档的 boost 后分数相加，`knn.filter` 只约束向量分支。多个顶层 `knn` 仍不支持。

## 容量和运维

容量评估必须使用真实数据和自然 segment 拓扑。HNSW 图按 segment 构建；过于频繁的 refresh 会产生大量小 segment，后台 merge
又会增加 CPU、I/O 和临时磁盘开销。force merge 后的单 segment 结果不能代表持续写入场景。

上线前至少验证：

| 项目 | 应记录的证据 |
| --- | --- |
| 检索效果 | Recall@K、NDCG 或业务标注指标 |
| 查询 | 并发 QPS、P50/P95/P99、错误率、查询 CPU |
| 写入 | Bulk 吞吐、失败项、refresh/merge 时间、GC |
| 存储 | store size、segment 数、分片和副本总占用 |
| 资源 | JVM heap、进程 RSS、文件系统缓存、磁盘水位和 thread-pool rejection |

不同数据分布、过滤选择性、维度、`m` 和 `num_candidates` 的结果不能直接互相替代。调参时一次只改变一个关键变量，并保留可复现的
mapping、语料、查询集和结果。

## 建模检查清单

- 确认新索引使用原生 HNSW 还是已有旧插件接口，不混用字段和查询语法；
- 冻结 Embedding 模型、版本、维度、归一化和相似度合同；
- 为每个 `dense_vector` 显式设置 `index: true`；如果省略 `index_options`，确认默认的 `hnsw(m=16, ef_construction=100)`
  符合效果和容量需求；
- 只为实际需要独立查询的语义视角建立向量字段；
- 设计同步或异步写入的失败恢复、模型版本和陈旧向量检测；
- 用生产预期的文档量、过滤条件、并发和 segment 拓扑验收效果、性能与容量；
- 模型或 mapping 合同变化时通过新字段或新索引迁移，并保留 alias 回滚路径。

## 相关章节

- [原生 HNSW 搜索]({{< relref "/docs/features/vector-search/native-hnsw.md" >}})
- [dense_vector 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/dense-vector.md" >}})
- [向量搜索与语义搜索]({{< relref "/docs/features/vector-search/vector-and-semantic-search.md" >}})
- [向量工作流]({{< relref "/docs/integrations/ai/vector-workflow.md" >}})
- [旧 k-NN 插件字段]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})

