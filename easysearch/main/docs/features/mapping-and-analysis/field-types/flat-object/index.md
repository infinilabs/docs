---
title: "扁平对象字段类型（Flat Object）"
date: 0001-01-01
summary: "扁平对象字段类型（Flat Object） #  flat_object（也称为 flattened 类型）将整个 JSON 对象作为单个扁平化字段存储。它会将 JSON 对象中所有叶子节点的值提取为关键字（keyword），并支持对这些值进行基本查询。
适用场景 #  flat_object 特别适合以下场景：
 标签/标注数据：如 Kubernetes 的 labels、annotations 等动态键值对 防止 mapping explosion：当字段名不可预知或数量极多时，避免为每个子字段创建独立的映射条目 半结构化 JSON：存储结构不固定的 JSON 数据，同时保持可搜索性 日志元数据：存储动态的日志属性或自定义元数据  与其他类型对比 #     类型 动态字段支持 查询能力 Mapping 条目 性能     object ✅ 每个子字段独立映射 完整（全文、范围等） 多（每子字段一条） 高（独立索引）   nested ✅ 每个子字段独立映射 完整 + 对象关联性 多 + 隐藏文档 中   flat_object ✅ 无需预定义 基本（term、prefix、range） 仅一条 中    创建映射 #  PUT my_index { &#34;mappings&#34;: { &#34;properties&#34;: { &#34;labels&#34;: { &#34;type&#34;: &#34;flat_object&#34; } } } } flat_object 使用默认配置即可，不需要额外的映射参数。"
---


# 扁平对象字段类型（Flat Object）

`flat_object`（也称为 flattened 类型）将整个 JSON 对象作为单个扁平化字段存储。它会将 JSON 对象中所有叶子节点的值提取为关键字（keyword），并支持对这些值进行基本查询。

## 适用场景

`flat_object` 特别适合以下场景：

- **标签/标注数据**：如 Kubernetes 的 labels、annotations 等动态键值对
- **防止 mapping explosion**：当字段名不可预知或数量极多时，避免为每个子字段创建独立的映射条目
- **半结构化 JSON**：存储结构不固定的 JSON 数据，同时保持可搜索性
- **日志元数据**：存储动态的日志属性或自定义元数据

## 与其他类型对比

| 类型 | 动态字段支持 | 查询能力 | Mapping 条目 | 性能 |
|------|-------------|---------|-------------|------|
| `object` | ✅ 每个子字段独立映射 | 完整（全文、范围等） | 多（每子字段一条） | 高（独立索引） |
| `nested` | ✅ 每个子字段独立映射 | 完整 + 对象关联性 | 多 + 隐藏文档 | 中 |
| `flat_object` | ✅ 无需预定义 | 基本（term、prefix、range） | **仅一条** | 中 |

## 创建映射

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "labels": {
        "type": "flat_object"
      }
    }
  }
}
```

`flat_object` 使用默认配置即可，不需要额外的映射参数。

## 索引文档

```json
PUT my_index/_doc/1
{
  "labels": {
    "env": "production",
    "team": "backend",
    "version": "2.1.0",
    "priority": "high"
  }
}

PUT my_index/_doc/2
{
  "labels": {
    "env": "staging",
    "team": "frontend",
    "version": "1.5.0",
    "region": "us-east-1"
  }
}
```

支持嵌套结构：

```json
PUT my_index/_doc/3
{
  "labels": {
    "deploy": {
      "region": "cn-north-1",
      "zone": "a"
    },
    "team": "infra"
  }
}
```

## 查询方式

### term 查询

按值查询（不指定路径，匹配任意键的值）：

```json
GET my_index/_search
{
  "query": {
    "term": {
      "labels": "production"
    }
  }
}
```

按路径.值查询（指定具体键）：

```json
GET my_index/_search
{
  "query": {
    "term": {
      "labels.env": "production"
    }
  }
}
```

嵌套路径查询：

```json
GET my_index/_search
{
  "query": {
    "term": {
      "labels.deploy.region": "cn-north-1"
    }
  }
}
```

### terms 查询

```json
GET my_index/_search
{
  "query": {
    "terms": {
      "labels.env": ["production", "staging"]
    }
  }
}
```

### prefix 查询

```json
GET my_index/_search
{
  "query": {
    "prefix": {
      "labels.version": "2."
    }
  }
}
```

### range 查询

```json
GET my_index/_search
{
  "query": {
    "range": {
      "labels.version": {
        "gte": "2.0.0",
        "lt": "3.0.0"
      }
    }
  }
}
```

> **注意**：range 查询基于字符串比较（字典序），不是数值比较。`"9"` 会大于 `"10"`。

### exists 查询

检查字段是否存在：

```json
GET my_index/_search
{
  "query": {
    "exists": {
      "field": "labels"
    }
  }
}
```

检查特定键是否存在：

```json
GET my_index/_search
{
  "query": {
    "exists": {
      "field": "labels.region"
    }
  }
}
```

## 聚合

`flat_object` 字段支持基于 doc values 的聚合：

```json
GET my_index/_search
{
  "size": 0,
  "aggs": {
    "env_values": {
      "terms": {
        "field": "labels.env"
      }
    }
  }
}
```

## 内部原理

`flat_object` 在内部为每个字段创建三个子字段：

| 子字段 | 存储内容 | 用途 |
|--------|---------|------|
| 父字段 | 所有 JSON 键名 | `exists` 查询 |
| `._value` | 所有叶子节点的值 | 不带路径的 term/prefix/range 查询 |
| `._valueAndPath` | `路径=值` 格式的条目 | 带路径的 term/prefix/range 查询 |

例如，文档 `{"labels": {"env": "prod", "team": "be"}}` 会被索引为：
- `._value`: `"prod"`, `"be"`
- `._valueAndPath`: `"env=prod"`, `"team=be"`

当查询 `labels.env = "prod"` 时，实际搜索的是 `._valueAndPath` 子字段中的 `"env=prod"`。

## 限制

- **不支持 wildcard 查询**：`flat_object` 字段不能使用 `wildcard` 查询
- **字符串比较**：所有值按关键字（keyword）处理，range 查询基于字典序而非数值
- **不支持全文搜索**：不能使用 `match`、`match_phrase` 等全文查询
- **不支持高亮**：查询结果中不支持高亮显示
- **不支持自定义分析器**：值始终按 keyword 处理，不进行分词
- **嵌套深度**：非常深的嵌套 JSON 会增加 `._valueAndPath` 条目的长度

## 最佳实践

1. **动态标签场景优选**：Kubernetes labels、自定义 tags 等字段名不固定的场景，用 `flat_object` 可以避免 mapping 膨胀
2. **不要用于需要全文搜索的字段**：如果需要分词搜索，应使用 `text` 类型
3. **不要用于需要数值运算的字段**：如果需要数值 range 或聚合运算，应使用 `integer`/`float` 等数值类型
4. **合理控制对象大小**：虽然 `flat_object` 没有子字段限制，但过大的 JSON 对象会影响索引性能

