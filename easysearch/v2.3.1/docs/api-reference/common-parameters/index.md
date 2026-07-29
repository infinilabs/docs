---
title: "常用 URL 参数"
date: 0001-01-01
summary: "常用 URL 参数 #  Easysearch 的 REST API 支持以下通用参数，适用于大部分接口。
   参数 说明 示例     pretty 以缩进格式返回 JSON，便于阅读和调试。 GET &lt;index_name&gt;/_search?pretty=true   human 将输出中的数值转换为人类可读格式（如 1h 代替 3600000ms，1kb 代替 1024 bytes）。 GET &lt;index_name&gt;/_search?human=true   error_trace 当请求出错时，在响应中包含完整的异常堆栈信息，便于排查问题。 GET &lt;index_name&gt;/_search?error_trace=true   format 指定返回格式。CAT API 支持 json、yaml、text 等格式。 GET _cat/indices?format=json   filter_path 过滤响应字段，只返回指定路径的内容，减少网络传输量。支持通配符 *。 GET _cluster/health?filter_path=status,number_of_nodes   flat_settings 以扁平化格式返回设置（index.number_of_shards 而非嵌套对象）。 GET &lt;index_name&gt;/_settings?flat_settings=true   Content-Type 请求头，指定请求体的内容类型。支持 application/json、application/yaml、application/cbor。 POST _scripts/&lt;template&gt; -H 'Content-Type: application/json'   source / source_content_type 当客户端不支持在非 POST 请求中发送请求体时，可通过查询参数传递请求体内容及其类型。 GET _search?"
---


# 常用 URL 参数

Easysearch 的 REST API 支持以下通用参数，适用于大部分接口。

| 参数                 | 说明                                                                                                                             | 示例                                                                                     |
| :------------------- | :------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------- |
| `pretty`             | 以缩进格式返回 JSON，便于阅读和调试。                                                                                             | `GET <index_name>/_search?pretty=true`                                                   |
| `human`              | 将输出中的数值转换为人类可读格式（如 `1h` 代替 3600000ms，`1kb` 代替 1024 bytes）。                                                | `GET <index_name>/_search?human=true`                                                    |
| `error_trace`        | 当请求出错时，在响应中包含完整的异常堆栈信息，便于排查问题。                                                                      | `GET <index_name>/_search?error_trace=true`                                              |
| `format`             | 指定返回格式。CAT API 支持 `json`、`yaml`、`text` 等格式。                                                                        | `GET _cat/indices?format=json`                                                           |
| `filter_path`        | 过滤响应字段，只返回指定路径的内容，减少网络传输量。支持通配符 `*`。                                                               | `GET _cluster/health?filter_path=status,number_of_nodes`                                 |
| `flat_settings`      | 以扁平化格式返回设置（`index.number_of_shards` 而非嵌套对象）。                                                                   | `GET <index_name>/_settings?flat_settings=true`                                          |
| `Content-Type`       | 请求头，指定请求体的内容类型。支持 `application/json`、`application/yaml`、`application/cbor`。                                    | `POST _scripts/<template> -H 'Content-Type: application/json'`                           |
| `source` / `source_content_type` | 当客户端不支持在非 POST 请求中发送请求体时，可通过查询参数传递请求体内容及其类型。                              | `GET _search?source_content_type=application/json&source={"query":{"match_all":{}}}` |

## 使用建议

- **开发调试**时建议加上 `?pretty=true`，生产环境去掉以减少响应体积
- **监控脚本**中使用 `filter_path` 只获取需要的字段，降低解析开销
- **排查异常**时加上 `error_trace=true` 获取完整错误堆栈

