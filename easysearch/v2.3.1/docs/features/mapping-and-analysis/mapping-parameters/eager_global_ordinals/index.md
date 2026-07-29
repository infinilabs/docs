---
title: "预加载全局序数参数（Eager Global Ordinals）"
date: 0001-01-01
summary: "Eager Global Ordinals 参数 #  eager_global_ordinals 参数控制是否在索引刷新时提前构建全局序数（Global Ordinals）。全局序数是 keyword、ip 等字段执行聚合和排序时使用的内部数据结构。
相关指南 #    doc_values 参数  fielddata 参数  参数选项 #     值 说明     false 默认值。全局序数在首次查询（聚合/排序）时惰性构建。   true 在索引刷新（refresh）时立即构建全局序数。    默认行为 #  默认情况下，全局序数采用惰性加载策略：第一次聚合或排序请求会触发构建，后续请求直接使用缓存。当索引发生变更并刷新后，缓存失效，下次查询时重新构建。
示例 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;category&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;eager_global_ordinals&#34;: true } } } } 何时启用 #     场景 建议     高频聚合字段，要求低延迟 设为 true，将构建开销转移到索引阶段   普通聚合字段 保持默认 false，惰性构建已足够   很少聚合的字段 保持默认 false，避免浪费资源   写入量远大于查询量 保持默认 false，频繁刷新会导致频繁重建    注意事项 #   启用后会增加索引刷新时间，因为每次 refresh 都需要重新构建全局序数 全局序数存储在堆内存中，高基数字段会占用较多内存 可通过 _stats API 查看全局序数占用的内存大小 对于时序数据等写入密集场景，建议保持默认值以避免刷新延迟增加  "
---


# Eager Global Ordinals 参数

`eager_global_ordinals` 参数控制是否在索引刷新时**提前构建全局序数**（Global Ordinals）。全局序数是 keyword、ip 等字段执行聚合和排序时使用的内部数据结构。

## 相关指南

- [doc_values 参数]({{< relref "/docs/features/mapping-and-analysis/mapping-parameters/doc_values.md" >}})
- [fielddata 参数]({{< relref "/docs/features/mapping-and-analysis/mapping-parameters/fielddata.md" >}})

## 参数选项

| 值 | 说明 |
|----|------|
| `false` | **默认值**。全局序数在首次查询（聚合/排序）时惰性构建。 |
| `true` | 在索引刷新（refresh）时立即构建全局序数。 |

## 默认行为

默认情况下，全局序数采用**惰性加载**策略：第一次聚合或排序请求会触发构建，后续请求直接使用缓存。当索引发生变更并刷新后，缓存失效，下次查询时重新构建。

## 示例

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "category": {
        "type": "keyword",
        "eager_global_ordinals": true
      }
    }
  }
}
```

## 何时启用

| 场景 | 建议 |
|------|------|
| 高频聚合字段，要求低延迟 | 设为 `true`，将构建开销转移到索引阶段 |
| 普通聚合字段 | 保持默认 `false`，惰性构建已足够 |
| 很少聚合的字段 | 保持默认 `false`，避免浪费资源 |
| 写入量远大于查询量 | 保持默认 `false`，频繁刷新会导致频繁重建 |

## 注意事项

- 启用后会**增加索引刷新时间**，因为每次 refresh 都需要重新构建全局序数
- 全局序数存储在**堆内存**中，高基数字段会占用较多内存
- 可通过 `_stats` API 查看全局序数占用的内存大小
- 对于时序数据等写入密集场景，建议保持默认值以避免刷新延迟增加

