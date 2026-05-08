---
title: "Point In Time (PIT) API"
date: 0001-01-01
summary: "Point In Time (PIT) API #  Point In Time（PIT）搜索具有与常规搜索相同的功能，不同之处在于 PIT 搜索作用于较旧的数据集，而常规搜索作用于实时数据集。PIT 搜索不绑定于特定查询，因此您可以在同一个冻结在时间点上的数据集上运行不同的查询。
您可以使用创建 PIT API 来创建 PIT。当您为一组索引创建 PIT 时，Easysearch 会锁定这些索引的一组段，使它们在时间上冻结。在底层，此 PIT 所需的资源不会被修改或删除。如果作为 PIT 一部分的段被合并，Easysearch 会在 PIT 创建时通过 keep_alive 参数指定的时间段内保留这些段的副本。
创建 PIT 操作会返回一个 PIT ID，您可以使用该 ID 在冻结的数据集上运行多个查询。即使索引继续摄取数据并修改或删除文档，PIT 引用的数据自 PIT 创建以来不会发生变化。当您的查询包含 PIT ID 时，您不需要将索引传递给搜索，因为它将使用该 PIT。使用 PIT ID 的搜索在多次运行时将产生完全相同的结果。
相关指南（先读这些） #    分页与排序  查询 DSL 基础  创建 PIT #  创建一个 PIT。查询参数 keep_alive 是必需的；它指定了保持 PIT 的时间长度。
端点 #  POST /&lt;target_indexes&gt;/_pit?"
---


# Point In Time (PIT) API

Point In Time（PIT）搜索具有与常规搜索相同的功能，不同之处在于 PIT 搜索作用于较旧的数据集，而常规搜索作用于实时数据集。PIT 搜索不绑定于特定查询，因此您可以在同一个冻结在时间点上的数据集上运行不同的查询。

您可以使用创建 PIT API 来创建 PIT。当您为一组索引创建 PIT 时，Easysearch 会锁定这些索引的一组段，使它们在时间上冻结。在底层，此 PIT 所需的资源不会被修改或删除。如果作为 PIT 一部分的段被合并，Easysearch 会在 PIT 创建时通过 `keep_alive` 参数指定的时间段内保留这些段的副本。

创建 PIT 操作会返回一个 PIT ID，您可以使用该 ID 在冻结的数据集上运行多个查询。即使索引继续摄取数据并修改或删除文档，PIT 引用的数据自 PIT 创建以来不会发生变化。当您的查询包含 PIT ID 时，您不需要将索引传递给搜索，因为它将使用该 PIT。使用 PIT ID 的搜索在多次运行时将产生完全相同的结果。

## 相关指南（先读这些）

- [分页与排序]({{< relref "/docs/features/fulltext-search/pagination-and-sorting.md" >}})
- [查询 DSL 基础]({{< relref "/docs/features/query-dsl/query-dsl-basics.md" >}})


## 创建 PIT

创建一个 PIT。查询参数 `keep_alive` 是必需的；它指定了保持 PIT 的时间长度。

### 端点

```json
POST /<target_indexes>/_pit?keep_alive=1h&routing=&expand_wildcards=&preference=
```

### 路径参数

参数 | 数据类型 | 描述
:--- | :--- | :---
target_indexes | 字符串 | PIT 的目标索引名称。可以包含以逗号分隔的列表或通配符索引模式。

### 查询参数

参数 | 数据类型 | 描述
:--- | :--- | :---
keep_alive | 时间 | 保持 PIT 的时间长度。每次使用搜索 API 访问 PIT 时，PIT 的生命周期都会延长一段等于 `keep_alive` 参数的时间。必需。
preference | 字符串 | 用于执行搜索的节点或分片。可选。默认为随机。
routing | 字符串 | 指定将搜索请求路由到特定分片。可选。默认为文档的 `_id`。
expand_wildcards | 字符串 | 可匹配通配符模式的索引类型。支持逗号分隔的值。有效值如下：<br>- `all`：匹配任何索引或数据流，包括隐藏的。<br>- `open`：匹配开放的、非隐藏的索引或非隐藏的数据流。<br>- `closed`：匹配关闭的、非隐藏的索引或非隐藏的数据流。<br>- `hidden`：匹配隐藏的索引或数据流。必须与 `open`、`closed` 或同时与 `open` 和 `closed` 组合使用。<br>- `none`：不接受通配符模式。<br>可选。默认为 `open`。
allow_partial_pit_creation | 布尔值 | 指定是否在部分失败的情况下创建 PIT。可选。默认为 `true`。


#### 请求示例

```json
POST /test-index/_pit?keep_alive=100m
```

#### 示例响应

```json
{
    "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFnNOWU43ckt3U3IyaFVpbGE1UWEtMncAFjFyeXBsRGJmVFM2RTB6eVg1aVVqQncAAAAAAAAAAAIWcDVrM3ZIX0pRNS1XejE5YXRPRFhzUQEWc05ZTjdyS3dTcjJoVWlsYTVRYS0ydwAA",
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "creation_time": 1658146050064
}
```

### 响应体字段

字段 | 数据类型                                                           | 描述
:--- |:---------------------------------------------------------------| :---
pit_id | [Base64 编码](../../mapping-and-analysis/field-types/binary.md) | PIT ID。
creation_time | 长整型                                                            | PIT 创建的时间，以毫秒为单位（从纪元开始）。



## 延长 PIT 时间

您可以在执行搜索时，通过在 `pit` 对象中提供 `keep_alive` 参数来延长 PIT 时间：

```json
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==",
    "keep_alive": "100m"
  },
  "sort": [
    {"@timestamp": {"order": "asc"}}
  ],
  "search_after": [
    "2021-05-20T05:30:04.832Z"
  ]
}
```

搜索请求中的 keep_alive 参数是可选的。它指定了延长保持 PIT 时间的时长。

## 列出所有 PIT

返回 Easysearch 集群中的所有 PIT。



#### 请求示例

```json
GET /_pit/_all
```

#### 示例响应

```json
{
    "pits": [
        {
            "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFnNOWU43ckt3U3IyaFVpbGE1UWEtMncAFjFyeXBsRGJmVFM2RTB6eVg1aVVqQncAAAAAAAAAAAEWcDVrM3ZIX0pRNS1XejE5YXRPRFhzUQEWc05ZTjdyS3dTcjJoVWlsYTVRYS0ydwAA",
            "creation_time": 1658146048666,
            "keep_alive": 6000000
        },
        {
            "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFnNOWU43ckt3U3IyaFVpbGE1UWEtMncAFjFyeXBsRGJmVFM2RTB6eVg1aVVqQncAAAAAAAAAAAIWcDVrM3ZIX0pRNS1XejE5YXRPRFhzUQEWc05ZTjdyS3dTcjJoVWlsYTVRYS0ydwAA",
            "creation_time": 1658146050064,
            "keep_alive": 6000000
        }
    ]
}
```

### 响应体字段

字段 | 数据类型 | 描述
:--- | :--- | :---
pits | JSON 对象数组 | 所有 PIT 的列表。

每个 PIT 对象包含以下字段。

字段 | 数据类型                                                                              | 描述
:--- |:----------------------------------------------------------------------------------| :---
pit_id | [Base64 编码](../../mapping-and-analysis/field-types/binary.md) | PIT ID。
creation_time | 长整型                                                                               | PIT 创建的时间，以毫秒为单位（从纪元开始）。
keep_alive | 长整型                                                                               | 保持 PIT 的时间长度，以毫秒为单位。

## 删除 PIT

删除一个、多个或所有 PIT。当 `keep_alive` 时间段过后，PIT 会自动删除。但是，为了释放资源，您可以使用删除 PIT API 来删除 PIT。删除 PIT API 支持通过 ID 删除 PIT 列表或一次性删除所有 PIT。


#### 请求示例: 删除所有 PIT

```json
DELETE /_pit/_all
```
如果您想删除一个或多个PIT，请在请求体中指定 PIT ID。

### 请求体字段

字段 | 数据类型                                                                                     | 描述
:--- |:-----------------------------------------------------------------------------------------| :---
pit_id | [Base64 编码](../../mapping-and-analysis/field-types/binary.md) 或二进制数组 | 要删除的 PIT 的 PIT ID。必需。

#### 示例请求：通过 ID 删除 PIT

```json
DELETE /_pit

{
    "pit_id": [
        "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAEWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA",
        "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAIWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA"
    ]
}
```

#### 示例响应

对于每个 PIT，响应包含一个带有 PIT ID 和 `successful` 字段的 JSON 对象，该字段指定删除是否成功。部分失败被视为失败。
```json
{
    "pits": [
        {
            "successful": true,
            "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAEWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA"
        },
        {
            "successful": false,
            "pit_id": "o463QQEPbXktaW5kZXgtMDAwMDAxFkhGN09fMVlPUkVPLXh6MUExZ1hpaEEAFjBGbmVEZHdGU1EtaFhhUFc4ZkR5cWcAAAAAAAAAAAIWaXBPNVJtZEhTZDZXTWFFR05waXdWZwEWSEY3T18xWU9SRU8teHoxQTFnWGloQQAA"
        }
    ]
}
```

### 响应体字段

字段 | 数据类型                                                           | 描述
:--- |:---------------------------------------------------------------| :---
successful | 布尔值                                                            | 删除操作是否成功。
pit_id | [Base64 编码](../../mapping-and-analysis/field-types/binary.md) | 要删除的 PIT 的 PIT ID。


## PIT 设置

您可以通过设置 `_cluaster/settings` 的方式为 PIT 指定以下设置。

设置 | 描述 | 默认值
:--- | :--- | :---
point_in_time.max_keep_alive | 一个集群级别的设置，指定 `keep_alive` 参数的最大值。 | 24h
search.max_open_pit_context | 一个节点级别的设置，指定节点的最大开放 PIT 上下文数量。 | 300

