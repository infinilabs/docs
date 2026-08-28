---
title: "dense_vector 字段类型"
date: 0001-01-01
description: "Easysearch 2.4.0 原生 HNSW dense_vector 字段的映射参数、约束和使用示例。"
summary: "dense_vector 字段类型 #  dense_vector 用于保存稠密浮点向量，并通过 Lucene 原生 HNSW 索引执行近似最近邻搜索。该能力从 Easysearch 2.4.0 开始内置，不需要安装 k-NN 插件。
旧 k-NN 插件的 knn_dense_float_vector 和 knn_sparse_bool_vector 字段仍然保留，参考 K-NN 向量字段类型。两套字段和查询语法不同， 新建原生 HNSW 索引时使用本页的 dense_vector。
创建映射 #  Easysearch 2.4.0 要求显式指定 dims、index: true 和 index_options.type: hnsw：
PUT /native-hnsw-demo { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;title&#34;: { &#34;type&#34;: &#34;text&#34; }, &#34;embedding&#34;: { &#34;type&#34;: &#34;dense_vector&#34;, &#34;dims&#34;: 4, &#34;element_type&#34;: &#34;float&#34;, &#34;index&#34;: true, &#34;similarity&#34;: &#34;cosine&#34;, &#34;index_options&#34;: { &#34;type&#34;: &#34;hnsw&#34;, &#34;m&#34;: 16, &#34;ef_construction&#34;: 100 } } } } } 映射参数 #     参数 是否必填 默认值 说明     type 是 - 必须为 dense_vector   dims 是 - 向量维度，取值范围为 1–4096   element_type 否 float 2."
---


# dense_vector 字段类型

`dense_vector` 用于保存稠密浮点向量，并通过 Lucene 原生 HNSW 索引执行近似最近邻搜索。该能力从 Easysearch 2.4.0
开始内置，不需要安装 k-NN 插件。

旧 k-NN 插件的 `knn_dense_float_vector` 和 `knn_sparse_bool_vector` 字段仍然保留，参考
[K-NN 向量字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/knn.md" >}})。两套字段和查询语法不同，
新建原生 HNSW 索引时使用本页的 `dense_vector`。

## 创建映射

Easysearch 2.4.0 要求显式指定 `dims`、`index: true` 和 `index_options.type: hnsw`：

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

## 映射参数

| 参数 | 是否必填 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `type` | 是 | - | 必须为 `dense_vector` |
| `dims` | 是 | - | 向量维度，取值范围为 1–4096 |
| `element_type` | 否 | `float` | 2.4.0 仅支持 `float` |
| `index` | 是 | - | 2.4.0 必须显式设置为 `true` |
| `similarity` | 否 | `cosine` | 向量相似度，取值见下表 |
| `index_options` | 是 | - | 2.4.0 必须显式指定原生 HNSW 配置 |
| `meta` | 否 | - | 字段元数据 |

`index_options` 支持以下参数：

| 参数 | 是否必填 | 默认值 | 取值范围 | 说明 |
| --- | --- | --- | --- | --- |
| `type` | 是 | - | `hnsw` | 原生 HNSW 索引类型 |
| `m` | 否 | `16` | 1–512 | 每个节点保留的最大连接数。增大该值通常提高召回率，同时增加内存、存储和构建成本 |
| `ef_construction` | 否 | `100` | 1–3200 | 构建图时使用的候选队列大小。增大该值通常提高图质量，同时降低写入速度 |

`m` 和 `ef_construction` 的取值范围由底层 Lucene HNSW vectors format 校验。mapping 解析阶段会保存这两个值，但不一定立即创建该 format；因此非法值可能不会在创建或更新 mapping 时报告，而会在首次写入包含该向量字段的文档、为该字段创建 segment writer 时失败。生产配置应在写入前使用有效范围。

未指定 `m` 和 `ef_construction` 时，可以使用简化写法：

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

## 字段能力

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

## 当前限制

Easysearch 2.4.0 的原生 HNSW 范围不包括：

- `byte`、`bit` 向量类型；
- `int8_hnsw`、`int4_hnsw`、`bbq_hnsw` 和 `flat` 索引类型；
- 省略 `dims` 后由首个文档推导维度；
- `index: false` 的非索引向量；
- nested kNN 查询。

查询方法参考[原生 HNSW 搜索]({{< relref "/docs/features/vector-search/native-hnsw.md" >}})。

