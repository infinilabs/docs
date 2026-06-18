---
title: "最小哈希分词过滤器（Min Hash）"
date: 0001-01-01
summary: "Min Hash 分词过滤器 #  min_hash 分词过滤器用于基于最小哈希近似算法为词元生成哈希值，这对于检测文档间的相似度很有用。min_hash 分词过滤器会为一组词元（通常来自一个经过分词的字段）生成哈希值。
相关指南（先读这些） #    文本分析：识别词元  文本分析：规范化  参数说明 #  最小哈希分词过滤器可以使用以下参数进行配置。
   参数 必需/可选 数据类型 描述     hash_count 可选 整数 为每个词元生成的哈希值数量。增加此值通常会提高相似度估计的准确性，但会增加计算成本。默认值为 1。   bucket_count 可选 整数 要使用的哈希桶数量。这会影响哈希的粒度。更多的桶能提供更细的粒度并减少哈希冲突，但需要更多内存。默认值为 512。   hash_set_size 可选 整数 每个桶中要保留的哈希数量。这会影响哈希质量。更大的集合大小可能会带来更好的相似度检测效果，但会消耗更多内存。默认值为 1。   with_rotation 可选 布尔值 当设置为 true 时，如果 hash_set_size 为 1，过滤器会用从其右侧按环形顺序找到的第一个非空桶的值填充空桶。如果 bucket_count 参数大于 1，此设置会自动默认为 true；否则，默认为 false。    参考样例 #  以下示例请求创建了一个名为 minhash_index 的新索引，并配置了一个带有最小哈希过滤器的分词器："
---


# Min Hash 分词过滤器

`min_hash` 分词过滤器用于基于最小哈希近似算法为词元生成哈希值，这对于检测文档间的相似度很有用。`min_hash` 分词过滤器会为一组词元（通常来自一个经过分词的字段）生成哈希值。

## 相关指南（先读这些）

- [文本分析：识别词元]({{< relref "/docs/features/mapping-and-analysis/text-analysis-identifying-words.md" >}})
- [文本分析：规范化]({{< relref "/docs/features/mapping-and-analysis/text-analysis-normalization.md" >}})

## 参数说明

最小哈希分词过滤器可以使用以下参数进行配置。

| 参数            | 必需/可选 | 数据类型 | 描述                                                                                                                                                                                         |
| --------------- | --------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `hash_count`    | 可选      | 整数     | 为每个词元生成的哈希值数量。增加此值通常会提高相似度估计的准确性，但会增加计算成本。默认值为 1。                                                                                             |
| `bucket_count`  | 可选      | 整数     | 要使用的哈希桶数量。这会影响哈希的粒度。更多的桶能提供更细的粒度并减少哈希冲突，但需要更多内存。默认值为 `512`。                                                                             |
| `hash_set_size` | 可选      | 整数     | 每个桶中要保留的哈希数量。这会影响哈希质量。更大的集合大小可能会带来更好的相似度检测效果，但会消耗更多内存。默认值为 1。                                                                     |
| `with_rotation` | 可选      | 布尔值   | 当设置为 `true` 时，如果 `hash_set_size` 为 1，过滤器会用从其右侧按环形顺序找到的第一个非空桶的值填充空桶。如果 `bucket_count` 参数大于 1，此设置会自动默认为 `true`；否则，默认为 `false`。 |

## 参考样例

以下示例请求创建了一个名为 `minhash_index` 的新索引，并配置了一个带有最小哈希过滤器的分词器：

```
PUT /minhash_index
{
  "settings": {
    "analysis": {
      "filter": {
        "minhash_filter": {
          "type": "min_hash",
          "hash_count": 3,
          "bucket_count": 512,
          "hash_set_size": 1,
          "with_rotation": false
        }
      },
      "analyzer": {
        "minhash_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "minhash_filter"
          ]
        }
      }
    }
  }
}
```

## 产生的词元

使用以下请求来检查使用该分词器生成的词元：

```
POST /minhash_index/_analyze
{
  "analyzer": "minhash_analyzer",
  "text": "Easysearch is very powerful."
}
```

返回内容中包含了生成的词元（这些词元没有直观的可读性，因为它们代表的是哈希值）：

```
{
  "tokens" : [
    {
      "token" : "\u0000\u0000㳠锯ੲ걌䐩䉵",
      "start_offset" : 0,
      "end_offset" : 27,
      "type" : "MIN_HASH",
      "position" : 0
    },
    {
      "token" : "\u0000\u0000㳠锯ੲ걌䐩䉵",
      "start_offset" : 0,
      "end_offset" : 27,
      "type" : "MIN_HASH",
      "position" : 0
    },
    ...
```

