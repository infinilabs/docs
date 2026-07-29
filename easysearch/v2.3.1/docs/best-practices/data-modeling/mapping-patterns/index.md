---
title: "Mapping 模式与最佳实践"
date: 0001-01-01
description: "实用映射模式：多字段、keyword/text 策略、数组、对象与归一化。"
summary: "Mapping 模式与最佳实践 #  在理解了 Mapping 基础之后，本页给出几种最常用、最实用的 Mapping 设计模式，帮助你避免常见坑。
复杂数据类型：数组、对象与内部对象 #  多值字段（数组） #  任何字段都可以包含多个值，以数组形式索引：
{ &#34;tag&#34;: [ &#34;search&#34;, &#34;nosql&#34; ] } 要点：
 数组中所有值必须是相同数据类型（不能混用日期和字符串） 如果通过索引数组创建新字段，Easysearch 会用第一个值的数据类型作为字段类型 数组在索引时被处理为“多值字段”，可以搜索，但无序 搜索时不能指定“第一个”或“最后一个”元素   注意：从 Easysearch 获取文档时，_source 中的数组顺序与索引时一致；但索引层面是无序的，可以把数组想象成“装在袋子里的值”。
 空字段 #  以下三种情况被认为是空字段，不会被索引：
{ &#34;null_value&#34;: null, &#34;empty_array&#34;: [], &#34;array_with_null_value&#34;: [ null ] } 在 Lucene 中不能存储 null 值，所以空字段等同于不存在。
内部对象（嵌套对象） #  JSON 支持嵌套对象，例如：
{ &#34;tweet&#34;: &#34;Easysearch is very flexible&#34;, &#34;user&#34;: { &#34;id&#34;: &#34;@johnsmith&#34;, &#34;gender&#34;: &#34;male&#34;, &#34;age&#34;: 26, &#34;name&#34;: { &#34;full&#34;: &#34;John Smith&#34;, &#34;first&#34;: &#34;John&#34;, &#34;last&#34;: &#34;Smith&#34; } } } Easysearch 会自动将内部对象映射为 object 类型，并在 properties 下列出内部字段："
---


# Mapping 模式与最佳实践

在理解了 Mapping 基础之后，本页给出几种最常用、最实用的 Mapping 设计模式，帮助你避免常见坑。

## 复杂数据类型：数组、对象与内部对象

### 多值字段（数组）

任何字段都可以包含多个值，以数组形式索引：

```json
{
  "tag": [ "search", "nosql" ]
}
```

要点：

- 数组中所有值必须是相同数据类型（不能混用日期和字符串）
- 如果通过索引数组创建新字段，Easysearch 会用第一个值的数据类型作为字段类型
- 数组在索引时被处理为“多值字段”，可以搜索，但**无序**
- 搜索时不能指定“第一个”或“最后一个”元素

> 注意：从 Easysearch 获取文档时，`_source` 中的数组顺序与索引时一致；但索引层面是无序的，可以把数组想象成“装在袋子里的值”。

### 空字段

以下三种情况被认为是空字段，不会被索引：

```json
{
  "null_value": null,
  "empty_array": [],
  "array_with_null_value": [ null ]
}
```

在 Lucene 中不能存储 `null` 值，所以空字段等同于不存在。

### 内部对象（嵌套对象）

JSON 支持嵌套对象，例如：

```json
{
  "tweet": "Easysearch is very flexible",
  "user": {
    "id": "@johnsmith",
    "gender": "male",
    "age": 26,
    "name": {
      "full": "John Smith",
      "first": "John",
      "last": "Smith"
    }
  }
}
```

Easysearch 会自动将内部对象映射为 `object` 类型，并在 `properties` 下列出内部字段：

```json
{
  "user": {
    "type": "object",
    "properties": {
      "id": { "type": "keyword" },
      "gender": { "type": "keyword" },
      "age": { "type": "integer" },
      "name": {
        "type": "object",
        "properties": {
          "full": { "type": "text" },
          "first": { "type": "text" },
          "last": { "type": "text" }
        }
      }
    }
  }
}
```

### 内部对象是如何索引的

Lucene 不理解内部对象，它只处理扁平化的键值对。Easysearch 会将文档转换为：

```json
{
  "tweet": [ "easysearch", "flexible", "very" ],
  "user.id": [ "@johnsmith" ],
  "user.gender": [ "male" ],
  "user.age": [ 26 ],
  "user.name.full": [ "john", "smith" ],
  "user.name.first": [ "john" ],
  "user.name.last": [ "smith" ]
}
```

可以通过字段名（如 `first`）或完整路径（如 `user.name.first`）引用内部字段。

### 内部对象数组的限制

考虑包含内部对象的数组：

```json
{
  "followers": [
    { "age": 35, "name": "Mary White" },
    { "age": 26, "name": "Alex Jones" },
    { "age": 19, "name": "Lisa Smith" }
  ]
}
```

这个文档会被扁平化为：

```json
{
  "followers.age": [ 19, 26, 35 ],
  "followers.name": [ "alex", "jones", "lisa", "smith", "mary", "white" ]
}
```

**问题**：`{age: 35}` 和 `{name: Mary White}` 之间的关联性丢失了，因为每个多值字段只是一包无序的值。

- 可以问：“有一个26岁的追随者吗？” ✓
- 但不能问：“是否有一个26岁且名字叫 Alex Jones 的追随者？” ✗

如果需要保持对象内部字段的关联性，应使用 `nested` 类型（详见「数据建模」章节）。

## 模式一：text + keyword 双字段（multi-fields）

适用场景：同一个字段既要用于全文检索，又要用于精确过滤/排序/聚合。

惯用设计：

```json
{
  "name": {
    "type": "text",
    "fields": {
      "keyword": {
        "type": "keyword"
      }
    }
  }
}
```

这样可以在查询中：

- 用 `match` 作用在 `name`（全文检索）
- 用 `term` / `terms` / 聚合作用在 `name.keyword`（精确匹配）

## 模式二：动态映射控制

默认情况下，Easysearch 会为遇到的新字段自动创建映射（动态映射）。可以通过 `dynamic` 设置控制：

- `true`：动态添加新字段（默认）
- `false`：忽略新字段（不索引，但会保存在 `_source`）
- `strict`：遇到新字段抛出异常

示例：根对象严格，但允许某个内部对象动态：

```json
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "title": { "type": "text" },
      "stash": {
        "type": "object",
        "dynamic": true
      }
    }
  }
}
```

## 模式三：动态模板（dynamic_templates）

使用 `dynamic_templates` 可以完全控制新字段的映射规则，例如：

```json
{
  "mappings": {
    "dynamic_templates": [
      {
        "es": {
          "match": "*_es",
          "match_mapping_type": "string",
          "mapping": {
            "type": "text",
            "analyzer": "spanish"
          }
        }
      },
      {
        "en": {
          "match": "*",
          "match_mapping_type": "string",
          "mapping": {
            "type": "text",
            "analyzer": "english"
          }
        }
      }
    ]
  }
}
```

> 注意：`match_mapping_type: "string"` 会匹配动态检测为字符串的字段。在动态映射中，字符串字段会被映射为 `text` 类型（带 `keyword` 子字段），所以这个模板会应用到这些字段上。

模板按顺序检测，第一个匹配的会被应用。

## 模式四：日期检测控制

默认情况下，Easysearch 会检测字符串是否像日期（如 `2014-01-01`），如果是就映射为 `date` 类型。这可能造成问题：

- 第一个文档：`{ "note": "2014-01-01" }` → `note` 被映射为 `date`
- 第二个文档：`{ "note": "Logged out" }` → 会报错，因为不是有效日期

可以通过 `date_detection: false` 关闭日期检测：

```json
{
  "mappings": {
    "date_detection": false
  }
}
```

## 模式五：规范化字段

很多业务字段需要统一格式后再用于搜索/聚合，例如：

- 地区名（“北京市”“北京”）
- 手机号（带/不带区号、间隔符号）
- 邮箱、URL 等

推荐做法：

- `city_raw`：原始输入（text，用于全文检索）
- `city_normalized`：规范化后的形态（keyword，用于过滤和聚合）

这样既保留了原始信息，又便于在查询与统计时避免噪音。

## 模式六：为时间序列设计时间字段

时间序列场景中，建议：

- 使用统一的 `@timestamp` 或类似命名
- 一致的时间格式（通常是 ISO8601 或 epoch_millis）
- 确保该字段为 `date` 类型，以便：
  - 范围过滤（range）
  - 时间聚合（date_histogram）

这样可以让可视化/监控/分析工具有统一约定。

## 常见反模式与踩坑

- **所有字符串一律 `text`：**
  - 结果：无法高效做过滤/聚合/排序，mapping 难以维护。
  - 建议：人类可读字段用 `text + keyword`，ID/code/枚举直接用 `keyword`。

- **在 `text` 字段上做 `term`/`range` 查询：**
  - 结果：匹配的是 analyzer 产出的 token，而不是原始文本，行为常常“看起来对、其实不对”。
  - 建议：全文统一走 `match`/`match_phrase`，精确条件落在 `keyword`/数值/日期字段。

- **需要子对象语义，却只用 `object` + 数组：**
  - 结果：内部字段关联性丢失，出现“26 岁 Mary White 被配成 19 岁 Lisa Smith”的假匹配。
  - 建议：一旦需要问“同一个子对象里字段是否同时满足”，就应建成 `nested`，并按数据建模章节建议设计查询。

- **放任动态映射自由生长：**
  - 结果：字段爆炸（`field1`、`field_1`、`field-1` 等各成一列），类型漂移，mapping 越来越难管。
  - 建议：对核心索引使用 `dynamic: strict` 或受控的 `dynamic_templates`，为“杂物袋”预留专门对象字段。

- **指望在线修改字段类型或分析器来“纠正历史”：**
  - 结果：硬改 mapping 不会重写历史数据，容易出现新旧数据行为不一致，甚至直接报错。
  - 建议：遇到类型/分析器设计错误时，优先采用“新索引 + 重建索引 + 别名切换”的套路。

## 小结

- 数组字段会被处理为多值字段，索引时无序；内部对象数组会丢失对象内部关联性，需要 `nested` 类型才能保持关联
- 使用 multi-fields（text + keyword）统一解决“模糊搜索 + 精确过滤/聚合”的组合需求
- 通过 `dynamic` 和 `dynamic_templates` 控制动态映射行为，避免意外字段类型推断
- 提前为聚合、排序和时间序列准备好类型正确、命名统一的字段
- 适度引入“规范化字段”来保证查询与聚合时的一致性

下一步可以继续阅读：

- [Query DSL 基础](../query-dsl/query-dsl-basics.md)
- [结构化搜索](../query-dsl/structured-search.md)
- [Nested](../../best-practices/data-modeling/nested)

## 参考手册（API 与参数）

- [字段类型总览（功能手册）]({{< relref "docs/features/mapping-and-analysis/field-types/_index.md" >}})
- [映射参数（功能手册）]({{< relref "docs/features/mapping-and-analysis/mapping-parameters/_index.md" >}})

