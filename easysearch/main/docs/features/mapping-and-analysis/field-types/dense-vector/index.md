---
title: "dense_vector 字段类型"
date: 0001-01-01
description: "dense_vector 普通 HNSW 与 Easysearch 3.0 OSQ HNSW 的映射参数、版本约束和使用示例。"
summary: "dense_vector 字段类型 #  dense_vector 用于保存稠密浮点向量，并通过 Lucene 原生 HNSW 索引执行近似最近邻搜索。该能力从 Easysearch 2.4.0 开始内置，不需要安装 k-NN 插件。
Easysearch 3.0 新增 OSQ HNSW（Optimized Scalar Quantization，优化的标量量化），通过 ES 兼容参数 index_options.type: &quot;bbq_hnsw&quot; 显式选择。省略 index_options 时仍使用普通 HNSW，不会自动选择量化类型。 本页先说明 2.4.0 起的普通 HNSW 配置；3.0 新增和变化的行为见下方OSQ HNSW 映射。
旧 k-NN 插件的 knn_dense_float_vector 和 knn_sparse_bool_vector 字段仍然保留，参考 K-NN 向量字段类型。两套字段和查询语法不同， 新建原生 HNSW 索引时使用本页的 dense_vector。
创建映射 #  Easysearch 2.4.0 要求显式指定 dims 和 index: true。以下示例省略 index_options：
PUT /native-hnsw-demo { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;embedding&#34;: { &#34;type&#34;: &#34;dense_vector&#34;, &#34;dims&#34;: 4, &#34;element_type&#34;: &#34;float&#34;, &#34;index&#34;: true, &#34;similarity&#34;: &#34;cosine&#34; } } } }  重要：省略 index_options 后仍会建立 HNSW 索引。 Easysearch 2."
---


# dense_vector 字段类型

`dense_vector` 用于保存稠密浮点向量，并通过 Lucene 原生 HNSW 索引执行近似最近邻搜索。该能力从 Easysearch 2.4.0
开始内置，不需要安装 k-NN 插件。

Easysearch 3.0 新增 **OSQ HNSW**（Optimized Scalar Quantization，优化的标量量化），通过 ES 兼容参数
`index_options.type: "bbq_hnsw"` 显式选择。省略 `index_options` 时仍使用普通 HNSW，不会自动选择量化类型。
本页先说明 2.4.0 起的普通 HNSW 配置；3.0 新增和变化的行为见下方[OSQ HNSW 映射](#30-osq-hnsw-映射)。

旧 k-NN 插件的 `knn_dense_float_vector` 和 `knn_sparse_bool_vector` 字段仍然保留，参考
[K-NN 向量字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})。两套字段和查询语法不同，
新建原生 HNSW 索引时使用本页的 `dense_vector`。

## 创建映射

Easysearch 2.4.0 要求显式指定 `dims` 和 `index: true`。以下示例省略 `index_options`：

```json
PUT /native-hnsw-demo
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "embedding": {
        "type": "dense_vector",
        "dims": 4,
        "element_type": "float",
        "index": true,
        "similarity": "cosine"
      }
    }
  }
}
```

> **重要：省略 `index_options` 后仍会建立 HNSW 索引。** Easysearch 2.4.0 默认使用
> `hnsw(m=16, ef_construction=100)`，并会在 mapping 响应中回显这组实际参数。该默认值使用未量化的 `float`
> 向量，与 Elasticsearch 8.19 默认的 `int8_hnsw` 存储不等价。

## 映射参数

| 参数 | 是否必填 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `type` | 是 | - | 必须为 `dense_vector` |
| `dims` | 是 | - | 向量维度，取值范围为 1–4096 |
| `element_type` | 否 | `float` | 2.4.0 仅支持 `float` |
| `index` | 是 | - | 2.4.0 必须显式设置为 `true` |
| `similarity` | 否 | `cosine` | 向量相似度，取值见下表 |
| `index_options` | 否 | `hnsw(m=16, ef_construction=100)` | 原生 HNSW 配置；需要调整构图参数时显式设置 |
| `meta` | 否 | - | 字段元数据 |

显式设置 `index_options` 时支持以下参数：

| 参数 | 是否必填 | 默认值 | 取值范围 | 说明 |
| --- | --- | --- | --- | --- |
| `type` | 是 | - | `hnsw` | 原生 HNSW 索引类型 |
| `m` | 否 | `16` | 1–512 | 每个节点保留的最大连接数。增大该值通常提高召回率，同时增加内存、存储和构建成本 |
| `ef_construction` | 否 | `100` | 1–3200 | 构建图时使用的候选队列大小。增大该值通常提高图质量，同时降低写入速度 |

`m` 和 `ef_construction` 的取值范围由底层 Lucene HNSW vectors format 校验。mapping 解析阶段会保存这两个值，但不一定立即创建该 format；因此非法值可能不会在创建或更新 mapping 时报告，而会在首次写入包含该向量字段的文档、为该字段创建 segment writer 时失败。生产配置应在写入前使用有效范围。

显式提供 `index_options` 但不需要自定义 `m` 和 `ef_construction` 时，也可以使用以下写法：

```json
"embedding": {
  "type": "dense_vector",
  "dims": 4,
  "index": true,
  "index_options": {
    "type": "hnsw"
  }
}
```

## 相似度

| `similarity` | 适用场景和约束 |
| --- | --- |
| `cosine` | 默认值。向量不能为零向量；Easysearch 在索引和查询时执行归一化，`_source` 保留原始值 |
| `dot_product` | 向量必须为单位向量，平方模长与 1 的误差不能超过 `1e-3` |
| `l2_norm` | 使用欧氏距离，适合未归一化的连续向量 |
| `max_inner_product` | 使用最大内积，允许向量幅值影响排序 |

所有向量元素必须是有限数值，不能包含 `NaN` 或无穷值。写入向量和查询向量的维度必须与 `dims` 完全一致。

## 写入向量

向量直接使用 JSON 数组写入：

```json
PUT /native-hnsw-demo/_doc/v01
{
  "title": "向量文档一",
  "embedding": [1.0, 0.0, 0.0, 0.0]
}
```

`index`、`create`、Bulk、Update 和 Reindex 写入都会执行相同的类型、维度和相似度约束。字段值为 `null` 时不会创建向量索引项。

## 版本和更新限制

- 只有使用 Easysearch 2.4.0 或更高版本创建的索引才能新增已索引的 `dense_vector` 字段。
- 旧索引不能通过更新 mapping 直接获得原生 HNSW 能力。请创建新索引并使用 Reindex 或迁移流程导入数据。
- 集群中存在 2.4.0 之前的数据节点时，包含已索引 `dense_vector` 的 mapping 和请求会被拒绝。

已存在字段的 mapping 更新规则如下：

| 参数 | 是否可更新 | 规则 |
| --- | --- | --- |
| `dims` | 否 | 维度变化需要新建索引并 Reindex |
| `element_type` | 否 | 2.4.0 只支持 `float` |
| `index` | 否 | 2.4.0 只支持显式的 `index: true` |
| `similarity` | 否 | 相似度变化需要新建索引并 Reindex |
| `index_options.type` | 否 | 2.4.0 只支持 `hnsw` |
| `index_options.m` | 只能增大 | 可以保持不变或增大，不能减小 |
| `index_options.ef_construction` | 是 | 可以增大或减小，但不能同时将 `m` 调小 |
| `meta` | 是 | 只更新字段元数据，不影响 HNSW 图 |

更新 `m` 或 `ef_construction` 不会重写已经存在的 segment，新的参数只用于更新后生成的 segment。因此同一索引可能暂时包含
使用不同 HNSW 构建参数的 segment。需要让全部数据使用统一参数时，请创建新索引并 Reindex；不要依赖 mapping 更新或
force merge 改写已有图。

## 2.4.0 字段能力

`dense_vector` 是单值、已索引但没有 doc values 的专用向量字段：

| 操作 | 支持情况 | 说明 |
| --- | --- | --- |
| 单个浮点向量 | 支持 | 数组长度必须等于 `dims` |
| 多值向量 | 不支持 | 同一文档的同一字段只能写入一个向量 |
| `null` | 支持 | 不为该文档创建向量索引项 |
| `exists` 查询 | 支持 | 可判断文档是否写入了该向量字段 |
| `term` 等普通字段查询 | 不支持 | 向量检索必须使用 `knn` |
| `_source` 返回 | 支持 | 返回写入时的原始数组 |
| 字段排序 | 不支持 | 该字段没有 doc values |
| 字段聚合 | 不支持 | 该字段没有 field data/doc values |
| `doc['embedding']` 脚本访问 | 不支持 | 需要在其他字段中保存可供脚本读取的派生值 |
| multi-fields | 不支持 | `dense_vector` 不能作为 multi-field 使用 |

使用 `cosine` 时，Easysearch 会将用于 HNSW 索引的向量归一化，但 `_source` 仍保留客户端写入的原始向量。读取 `_source`
得到的值不一定等于底层索引使用的归一化值。

## 2.4.0 的限制

Easysearch 2.4.0 的原生 HNSW 范围不包括：

- `byte`、`bit` 向量类型；
- `int8_hnsw`、`int4_hnsw`、`bbq_hnsw` 和 `flat` 索引类型；
- 省略 `dims` 后由首个文档推导维度；
- `index: false` 的非索引向量；
- nested kNN 查询。

查询方法参考[原生 HNSW 搜索]({{< relref "/docs/features/vector-search/native-hnsw.md" >}})。

## 3.0 OSQ HNSW 映射

以下配置用于具备 OSQ 能力的 Easysearch 3.0 集群中新建的索引，不适用于 2.x：

```json
PUT /osq-vectors
{
  "settings": {
    "index.mapping.exclude_source_vectors": true
  },
  "mappings": {
    "properties": {
      "vector": {
        "type": "dense_vector",
        "dims": 1024,
        "index": true,
        "similarity": "cosine",
        "index_options": {
          "type": "bbq_hnsw",
          "m": 64,
          "ef_construction": 512
        }
      }
    }
  }
}
```

| 参数或能力 | 3.0 显式 OSQ HNSW 行为 |
| --- | --- |
| `dims` | 64–4096；可省略，由首个有效向量确定，生产环境建议显式指定 |
| `element_type` | 默认 `float`，也支持 `bfloat16` |
| `index` | 省略时为 `true`，不接受 `false` |
| `similarity` | 默认 `cosine`；同时支持 `dot_product`、`l2_norm`、`max_inner_product` |
| `index_options.type` | 使用兼容 API 值 `bbq_hnsw`；不接受 `osq_hnsw` |
| `m` / `ef_construction` | 默认 16 / 100，范围与普通 HNSW 相同 |
| `index_options.rescore_vector.oversample` | 新 OSQ 字段默认 3；可设 0 关闭原精度重排，非零范围为 [1,10] |
| `index_options.on_disk_rescore` | 默认 `false`；控制 raw 读取策略，两种取值都保留 raw |
| `index_options.flat_index_threshold` | 默认 300；负值回落到 300，0 禁用小段省图优化 |
| nested | 支持 nested kNN 和 inner_hits |
| 字段读取 | 支持 `fields`、`docvalue_fields` 及向量脚本访问，仍不支持排序和聚合 |
| `_source` | 新 OSQ 索引默认自动排除向量，显式请求时从保留的原精度向量恢复 |

上面的默认行为受新索引语义和节点能力约束，不能仅凭版本号套用到旧索引。普通 HNSW 仍要求显式 `dims` 和
`index: true`；3.0 也接受显式 `index: false` 的非索引向量，但不能同时配置 `similarity` 或 `index_options`，不能执行 kNN 检索。

更新 OSQ 字段时，维度、元素类型和相似度不能改变，`m` 只能保持或增大。普通 HNSW 可显式转换为 OSQ HNSW，
反向转换不支持；mapping 更新只影响后续生成的段，不会立即将旧段全部改写为量化格式。

新 OSQ 索引默认 `index.mapping.exclude_source_vectors: true`，该设置只能在建索引时确定。物理 `_source` 省略向量数组，
Lucene 段内仍保留 raw，用于原精度重排、字段读取和 source 恢复；恢复不是从 1-bit 码反量化。
搜索需要返回向量时指定 `_source: {"exclude_vectors": false}`，GET 等入口可用 URL 参数 `_source_exclude_vectors=false`。
恢复值遵循保留的 float32／bfloat16 精度；需要在物理 `_source` 中保留输入数组时，应在建索引时关闭自动排除。

完整的写入、两种查询入口、候选重排、向量取回和迁移边界见
[OSQ HNSW 搜索]({{< relref "/docs/features/vector-search/osq-hnsw.md" >}})。

