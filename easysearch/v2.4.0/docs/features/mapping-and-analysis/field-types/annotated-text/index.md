---
title: "标注文本字段类型（Annotated Text）"
date: 0001-01-01
summary: "标注文本字段类型（Annotated Text） #  annotated_text 字段类型是 text 字段的扩展，允许在文本中嵌入标注标记（annotation tokens）。标注标记会在索引时注入到文本的词元流（token stream）中，可以像普通词语一样被搜索。
适用场景 #   命名实体识别（NER）：将文本中识别出的实体（人名、地名、组织名等）作为标注嵌入 文本分类标记：在文本中标记特定类别或主题 知识图谱关联：将文本中的实体链接到知识图谱的节点 自定义标签注入：在索引时为文本添加额外的可搜索标签  前置条件 #  annotated_text 字段类型由 mapper-annotated-text 插件提供，需要确认插件已安装：
bin/easysearch-plugin list # 应包含 mapper-annotated-text 创建映射 #  PUT my_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;content&#34;: { &#34;type&#34;: &#34;annotated_text&#34; } } } } annotated_text 支持与 text 字段相同的映射参数，如 analyzer、search_analyzer 等。
标注语法 #  标注使用特殊的标记语法嵌入在文本中。标注的格式为：
[被标注的文本](标注值) 示例 #  原始文本：
&#34;昨天在北京召开了搜索技术大会&#34; 带标注的文本：
&#34;昨天在[北京](LOC=北京&amp;wiki=Q956)召开了[搜索技术大会](EVENT=搜索技术大会)&#34; 索引带标注的文档 #  PUT my_index/_doc/1 { &#34;content&#34;: &#34;昨天[极限实验室](ORG=极限实验室)发布了[Easysearch](PRODUCT=Easysearch) 的新版本&#34; } PUT my_index/_doc/2 { &#34;content&#34;: &#34;[张三](PERSON=张三)在[上海](LOC=上海)参加了技术峰会&#34; } 查询 #  搜索标注值 #  标注值作为词元注入到了索引中，可以直接搜索："
---


# 标注文本字段类型（Annotated Text）

`annotated_text` 字段类型是 `text` 字段的扩展，允许在文本中嵌入标注标记（annotation tokens）。标注标记会在索引时注入到文本的词元流（token stream）中，可以像普通词语一样被搜索。

## 适用场景

- **命名实体识别（NER）**：将文本中识别出的实体（人名、地名、组织名等）作为标注嵌入
- **文本分类标记**：在文本中标记特定类别或主题
- **知识图谱关联**：将文本中的实体链接到知识图谱的节点
- **自定义标签注入**：在索引时为文本添加额外的可搜索标签

## 前置条件

`annotated_text` 字段类型由 `mapper-annotated-text` 插件提供，需要确认插件已安装：

```bash
bin/easysearch-plugin list
# 应包含 mapper-annotated-text
```

## 创建映射

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "annotated_text"
      }
    }
  }
}
```

`annotated_text` 支持与 `text` 字段相同的映射参数，如 `analyzer`、`search_analyzer` 等。

## 标注语法

标注使用特殊的标记语法嵌入在文本中。标注的格式为：

```
[被标注的文本](标注值)
```

### 示例

原始文本：

```
"昨天在北京召开了搜索技术大会"
```

带标注的文本：

```
"昨天在[北京](LOC=北京&wiki=Q956)召开了[搜索技术大会](EVENT=搜索技术大会)"
```

## 索引带标注的文档

```json
PUT my_index/_doc/1
{
  "content": "昨天[极限实验室](ORG=极限实验室)发布了[Easysearch](PRODUCT=Easysearch) 的新版本"
}

PUT my_index/_doc/2
{
  "content": "[张三](PERSON=张三)在[上海](LOC=上海)参加了技术峰会"
}
```

## 查询

### 搜索标注值

标注值作为词元注入到了索引中，可以直接搜索：

```json
GET my_index/_search
{
  "query": {
    "match": {
      "content": "PRODUCT=Easysearch"
    }
  }
}
```

### 搜索原始文本

普通文本搜索照常工作：

```json
GET my_index/_search
{
  "query": {
    "match": {
      "content": "新版本"
    }
  }
}
```

### 组合搜索

可以同时搜索原始文本和标注值：

```json
GET my_index/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "content": "发布" } },
        { "match": { "content": "ORG=极限实验室" } }
      ]
    }
  }
}
```

## 工作原理

当索引包含标注的文本时，`annotated_text` 字段会：

1. 对方括号中的被标注文本进行正常分词和索引
2. 将圆括号中的标注值作为额外的词元注入到相同的位置（position）
3. 标注值与被标注文本共享相同的位置偏移，因此短语查询可以同时匹配两者

例如文本 `"[北京](LOC=北京)天气"` 索引后的词元流：

| 位置 | 词元 |
|------|------|
| 0 | `北京`（原始文本分词） |
| 0 | `LOC=北京`（标注值，同一位置） |
| 1 | `天气` |

## 限制

- 需要安装 `mapper-annotated-text` 插件
- 标注语法必须严格遵循 `[文本](标注)` 格式
- 标注值不会经过分析器处理，作为完整的词元索引
- 大量标注会增加索引体积和索引时间

