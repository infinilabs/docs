---
title: "位置增量间隔参数（Position Increment Gap）"
date: 0001-01-01
summary: "Position Increment Gap 参数 #  position_increment_gap 参数控制数组中相邻文本值之间的位置间隔。该参数影响短语查询（match_phrase）在跨数组元素时的匹配行为。
相关指南 #    全文搜索  analyzer 参数  默认值 #  默认值为 100。
为什么需要 position_increment_gap #  当一个 text 字段接受数组值时，各元素的文本会被拼接分析。例如：
PUT my-index/_doc/1 { &#34;tags&#34;: [&#34;quick brown&#34;, &#34;fox jumps&#34;] } 分析后的词条位置：
   词条 位置     quick 0   brown 1   fox 102 （1 + 100 + 1）   jumps 103    默认间隔 100 使得 match_phrase 查询 &quot;brown fox&quot; 不会匹配该文档，因为 brown（位置 1）和 fox（位置 102）之间有 100 个位置的间隔，远大于短语查询允许的 slop。"
---


# Position Increment Gap 参数

`position_increment_gap` 参数控制数组中相邻文本值之间的**位置间隔**。该参数影响短语查询（`match_phrase`）在跨数组元素时的匹配行为。

## 相关指南

- [全文搜索]({{< relref "/docs/features/fulltext-search/" >}})
- [analyzer 参数]({{< relref "/docs/features/mapping-and-analysis/mapping-parameters/analyzer.md" >}})

## 默认值

默认值为 **100**。

## 为什么需要 position_increment_gap

当一个 `text` 字段接受数组值时，各元素的文本会被拼接分析。例如：

```json
PUT my-index/_doc/1
{
  "tags": ["quick brown", "fox jumps"]
}
```

分析后的词条位置：

| 词条 | 位置 |
|------|------|
| quick | 0 |
| brown | 1 |
| fox | **102** （1 + 100 + 1） |
| jumps | 103 |

默认间隔 100 使得 `match_phrase` 查询 `"brown fox"` **不会匹配**该文档，因为 `brown`（位置 1）和 `fox`（位置 102）之间有 100 个位置的间隔，远大于短语查询允许的 slop。

## 示例

### 允许跨元素匹配

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "tags": {
        "type": "text",
        "position_increment_gap": 0
      }
    }
  }
}
```

设为 0 后，`"brown fox"` 的短语查询将跨元素匹配。

### 自定义间隔

```json
PUT my-index
{
  "mappings": {
    "properties": {
      "names": {
        "type": "text",
        "position_increment_gap": 50
      }
    }
  }
}
```

## 何时调整

| 场景 | 建议 |
|------|------|
| 严格避免跨数组元素匹配 | 保持默认值 100 |
| 允许跨元素短语匹配 | 设为 0 |
| 需要精细控制跨元素匹配距离 | 设为合适的正整数 |

## 注意事项

- 仅对 `text` 类型字段的**数组值**有意义
- 如果字段不存储数组值，该参数无影响
- 与 `match_phrase` 查询的 `slop` 参数配合使用

