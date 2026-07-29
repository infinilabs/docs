---
title: "忽略超长参数（Ignore Above）"
date: 0001-01-01
summary: "Ignore Above 参数 #  ignore_above 参数指定字符串的最大长度限制。超过此长度的字符串不会被索引或存储在 doc_values 中，但仍会出现在 _source 中。
该参数主要用于 keyword 字段类型，防止超长字符串占用过多索引空间。
相关指南（先读这些） #    映射基础  Keyword 字段类型  参数选项 #     值 说明     正整数 超过此字符数的值不被索引。   默认值 2147483647（几乎不限制）。动态映射创建的 keyword 子字段默认为 256。    示例 #  PUT my-index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;tag&#34;: { &#34;type&#34;: &#34;keyword&#34;, &#34;ignore_above&#34;: 100 } } } } 写入测试：
PUT my-index/_doc/1 { &#34;tag&#34;: &#34;short_tag&#34; } PUT my-index/_doc/2 { &#34;tag&#34;: &#34;this_is_a_very_long_tag_value_that_exceeds_one_hundred_characters_and_therefore_should_not_be_indexed_at_all_in_the_inverted_index&#34; }  文档 1 的 tag 会被正常索引，可以搜索和聚合 文档 2 的 tag 不会被索引，无法通过 term 查询找到，也不会出现在聚合结果中，但仍然存在于 _source 中  动态映射的默认行为 #  当 Easysearch 动态检测到字符串字段时，会自动创建如下映射："
---


# Ignore Above 参数

`ignore_above` 参数指定字符串的最大长度限制。超过此长度的字符串**不会被索引或存储在 doc_values 中**，但仍会出现在 `_source` 中。

该参数主要用于 `keyword` 字段类型，防止超长字符串占用过多索引空间。

## 相关指南（先读这些）

- [映射基础]({{< relref "/docs/features/mapping-and-analysis/mapping-basics.md" >}})
- [Keyword 字段类型]({{< relref "/docs/features/mapping-and-analysis/field-types/string-field-type/keyword" >}})

## 参数选项

| 值 | 说明 |
|----|------|
| 正整数 | 超过此字符数的值不被索引。 |
| 默认值 | `2147483647`（几乎不限制）。动态映射创建的 keyword 子字段默认为 `256`。 |

## 示例

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "tag": {
        "type": "keyword",
        "ignore_above": 100
      }
    }
  }
}
```

写入测试：

```json
PUT my-index/_doc/1
{
  "tag": "short_tag"
}

PUT my-index/_doc/2
{
  "tag": "this_is_a_very_long_tag_value_that_exceeds_one_hundred_characters_and_therefore_should_not_be_indexed_at_all_in_the_inverted_index"
}
```

- 文档 1 的 `tag` 会被正常索引，可以搜索和聚合
- 文档 2 的 `tag` **不会被索引**，无法通过 term 查询找到，也不会出现在聚合结果中，但仍然存在于 `_source` 中

## 动态映射的默认行为

当 Easysearch 动态检测到字符串字段时，会自动创建如下映射：

```json
"my_field": {
  "type": "text",
  "fields": {
    "keyword": {
      "type": "keyword",
      "ignore_above": 256
    }
  }
}
```

这意味着超过 256 个字符的值不会在 keyword 子字段中被索引。如果你的数据中有超长字段值（如文章正文、日志消息），需要调整这个值：

- 需要对长字符串做精确匹配或聚合 → 增大 `ignore_above`
- 字段值确实很短（如标签、状态码） → 使用默认或更小的值
- 完全不需要 keyword 子字段 → 在映射中去掉 `fields.keyword`

## 与 index: false 的区别

| 特性 | `ignore_above` | `index: false` |
|------|----------------|----------------|
| 作用范围 | 仅跳过超长值 | 跳过所有值 |
| 短值 | 正常索引 | 也不索引 |
| _source | 所有值都保留 | 所有值都保留 |
| 适用类型 | keyword | 任意字段类型 |

## 注意事项

- `ignore_above` 统计的是**字符数**，不是字节数。对于中文等多字节字符，一个中文字符算一个字符
- 被忽略的值仍然保存在 `_source` 中，`_source` 的大小不受影响
- 该参数可以通过 `PUT mapping` API 动态更新（无需重建索引）

