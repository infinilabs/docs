---
title: "Fast Terms 查询"
date: 0001-01-01
summary: "Fast Terms 查询 #  Fast Terms 是 Easysearch 提供的高性能 terms 查询插件，专为大规模 terms 过滤场景优化。当需要在查询中使用大量 term 值进行过滤时（如数万甚至数十万个 ID），Fast Terms 可以提供比标准 terms 查询更好的性能。
适用场景 #   大规模 ID 过滤：根据外部系统提供的大量 ID 列表过滤文档 权限过滤：根据用户可访问的文档 ID 集合进行过滤 黑/白名单：根据预定义的大规模列表进行包含/排除过滤 高基数字段过滤：在高基数数值字段上进行大量值的匹配  工作原理 #  Fast Terms 使用 RoaringBitmap 和零分配哈希（XXHash）等技术对大规模 terms 集合进行压缩和高效匹配，相比标准 terms 查询在以下方面有显著优势：
   对比项 标准 terms 查询 Fast Terms     内存占用 较高（每个 term 独立存储） 低（Bitmap 压缩）   大规模 terms 性能 随 terms 数量线性下降 保持稳定   缓存效率 一般 高（LZ4 压缩 + 过期缓存）    使用方法 #  Bitmap Value Type #  在 terms 查询中使用 value_type: bitmap 来启用 Fast Terms 优化："
---


# Fast Terms 查询

Fast Terms 是 Easysearch 提供的高性能 terms 查询插件，专为大规模 terms 过滤场景优化。当需要在查询中使用大量 term 值进行过滤时（如数万甚至数十万个 ID），Fast Terms 可以提供比标准 terms 查询更好的性能。

## 适用场景

- **大规模 ID 过滤**：根据外部系统提供的大量 ID 列表过滤文档
- **权限过滤**：根据用户可访问的文档 ID 集合进行过滤
- **黑/白名单**：根据预定义的大规模列表进行包含/排除过滤
- **高基数字段过滤**：在高基数数值字段上进行大量值的匹配

## 工作原理

Fast Terms 使用 RoaringBitmap 和零分配哈希（XXHash）等技术对大规模 terms 集合进行压缩和高效匹配，相比标准 terms 查询在以下方面有显著优势：

| 对比项 | 标准 terms 查询 | Fast Terms |
|--------|----------------|------------|
| 内存占用 | 较高（每个 term 独立存储） | 低（Bitmap 压缩） |
| 大规模 terms 性能 | 随 terms 数量线性下降 | 保持稳定 |
| 缓存效率 | 一般 | 高（LZ4 压缩 + 过期缓存） |

## 使用方法

### Bitmap Value Type

在 terms 查询中使用 `value_type: bitmap` 来启用 Fast Terms 优化：

```json
GET my_index/_search
{
  "query": {
    "terms": {
      "user_id": {
        "value": [1001, 1002, 1003, 1005, 1008],
        "value_type": "bitmap"
      }
    }
  }
}
```

### 从索引加载 Terms

可以从另一个索引中动态加载 terms 值：

```json
GET my_index/_search
{
  "query": {
    "terms": {
      "user_id": {
        "index": "allowed_users",
        "id": "access_list_1",
        "path": "user_ids",
        "value_type": "bitmap"
      }
    }
  }
}
```

## 重新加载数据

Fast Terms 支持通过 API 重新加载缓存的数据：

```json
POST _fast_terms/_reload
```

## 注意事项

1. **字段类型**：Fast Terms 主要针对数值类型字段（`integer`、`long` 等）优化
2. **插件安装**：Fast Terms 作为插件提供，需要在所有数据节点上安装
3. **缓存管理**：Fast Terms 使用带过期时间的缓存，可自动释放不再使用的 bitmap 数据
4. **适用规模**：当 terms 数量较少（< 100）时，标准 terms 查询即可满足需求，无需使用 Fast Terms

