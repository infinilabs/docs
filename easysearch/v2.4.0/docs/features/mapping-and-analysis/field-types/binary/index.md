---
title: "二进制字段类型（Binary）"
date: 0001-01-01
summary: "二进制字段类型 #  二进制字段类型包含以 Base64 编码存储的二进制值，这些值不可被搜索。
参考代码 #  创建包含二进制字段的映射
PUT testindex { &#34;mappings&#34; : { &#34;properties&#34; : { &#34;binary_value&#34; : { &#34;type&#34; : &#34;binary&#34; } } } } 索引一个二进制值的文档
PUT testindex/_doc/1 { &#34;binary_value&#34; : &#34;bGlkaHQtd29rfx4=&#34; }  使用 = 作为填充字符。不允许嵌入换行符。
 参数说明 #  以下参数均为可选参数
   参数 数据类型 默认值 描述     doc_values Boolean false 是否存储在磁盘上，用于聚合、排序或脚本。   store Boolean false 是否单独存储字段值，使其可以独立于 _source 被检索。    使用场景 #  二进制字段适合存储小型二进制数据，例如："
---


# 二进制字段类型

二进制字段类型包含以 Base64 编码存储的二进制值，这些值不可被搜索。

## 参考代码

创建包含二进制字段的映射

```
PUT testindex
{
  "mappings" : {
    "properties" :  {
      "binary_value" : {
        "type" : "binary"
      }
    }
  }
}
```

索引一个二进制值的文档

```
PUT testindex/_doc/1
{
  "binary_value" : "bGlkaHQtd29rfx4="
}
```

> 使用 = 作为填充字符。不允许嵌入换行符。

## 参数说明

以下参数均为可选参数

| 参数          | 数据类型 | 默认值  | 描述                                                                  |
| ------------- | -------- | ------- | --------------------------------------------------------------------- |
| `doc_values`  | Boolean  | `false` | 是否存储在磁盘上，用于聚合、排序或脚本。                               |
| `store`       | Boolean  | `false` | 是否单独存储字段值，使其可以独立于 `_source` 被检索。                   |

## 使用场景

二进制字段适合存储小型二进制数据，例如：

- 文件的 MD5/SHA 摘要
- 小图标或缩略图
- 证书或密钥的 Base64 编码

> **注意**：二进制字段 **不可搜索**，不会建立倒排索引。它只是被存储在 `_source` 中随文档一起返回。

### 完整示例

```
PUT attachments
{
  "mappings": {
    "properties": {
      "filename": { "type": "keyword" },
      "content_type": { "type": "keyword" },
      "data": { "type": "binary" }
    }
  }
}
```

```
PUT attachments/_doc/1
{
  "filename": "logo.png",
  "content_type": "image/png",
  "data": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg=="
}
```

> **提示**：对于大文件，建议使用外部存储（如对象存储），在 Easysearch 中只存储文件的引用路径。

