---
title: "元数据字段"
date: 0001-01-01
description: "Easysearch 内置的元数据字段，用于文档标识、路由和调试"
summary: "元数据字段 #  元数据字段是 Easysearch 为每个文档自动维护的内部字段，用于文档标识、分片路由、原始数据存储等系统功能。
 元数据字段分类 #  文档标识 #     字段 说明 可配置      _id 文档唯一标识符 ✓    _index 文档所属索引名 -    路由控制 #     字段 说明 可配置      _routing 自定义分片路由值 ✓    数据存储 #     字段 说明 可配置      _source 文档原始 JSON ✓    _meta 用户自定义元数据 ✓    调试与审计 #     字段 说明 可配置      _ignored 被忽略的字段列表 -    _field_names 非空字段名列表 ✓     常用元数据字段详解 #  _id — 文档标识符 #  每个文档都有一个唯一的 _id，可以在索引时指定或由系统自动生成。"
---


# 元数据字段

元数据字段是 Easysearch 为每个文档自动维护的内部字段，用于文档标识、分片路由、原始数据存储等系统功能。

---

## 元数据字段分类

### 文档标识

| 字段 | 说明 | 可配置 |
|------|------|:------:|
| [_id](id.md) | 文档唯一标识符 | ✓ |
| [_index](index-field.md) | 文档所属索引名 | - |

### 路由控制

| 字段 | 说明 | 可配置 |
|------|------|:------:|
| [_routing](routing.md) | 自定义分片路由值 | ✓ |

### 数据存储

| 字段 | 说明 | 可配置 |
|------|------|:------:|
| [_source](source.md) | 文档原始 JSON | ✓ |
| [_meta](meta.md) | 用户自定义元数据 | ✓ |

### 调试与审计

| 字段 | 说明 | 可配置 |
|------|------|:------:|
| [_ignored](ignored.md) | 被忽略的字段列表 | - |
| [_field_names](field-names.md) | 非空字段名列表 | ✓ |

---

## 常用元数据字段详解

### _id — 文档标识符

每个文档都有一个唯一的 `_id`，可以在索引时指定或由系统自动生成。

```json
PUT /my-index/_doc/my-custom-id
{
  "title": "Hello World"
}
```

查询时可以直接使用 `_id`：

```json
GET /my-index/_doc/my-custom-id
```

### _source — 原始文档

`_source` 字段存储了文档索引时的原始 JSON，是搜索结果返回的数据来源。

可以在映射中配置是否存储或过滤 `_source`：

```json
PUT /my-index
{
  "mappings": {
    "_source": {
      "enabled": true,
      "excludes": ["password", "internal_*"]
    }
  }
}
```

### _routing — 分片路由

默认情况下，文档根据 `_id` 的哈希值分配到分片。使用自定义 `_routing` 可以控制相关文档存储在同一分片。

```json
PUT /my-index/_doc/1?routing=user123
{
  "user": "user123",
  "message": "Hello"
}
```

**使用场景**：
- 用户数据按 `user_id` 路由，提升单用户查询性能
- 父子文档必须使用相同路由

### _meta — 自定义元数据

`_meta` 字段用于存储与索引相关的自定义信息，如版本号、所有者等：

```json
PUT /my-index
{
  "mappings": {
    "_meta": {
      "version": "1.0",
      "owner": "data-team",
      "description": "用户行为日志索引"
    }
  }
}
```

---

## 元数据字段完整列表

| 字段 | 描述 |
|------|------|
| [_id](id.md) | 文档的唯一标识符 |
| [_index](index-field.md) | 文档所属的索引名称 |
| [_routing](routing.md) | 自定义分片路由值 |
| [_source](source.md) | 文档的原始 JSON 内容 |
| [_meta](meta.md) | 用户定义的任意元数据 |
| [_ignored](ignored.md) | 因映射限制被忽略的字段列表 |
| [_field_names](field-names.md) | 文档中所有非空字段的名称 |
