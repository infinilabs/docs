---
title: "Python 客户端"
date: 0001-01-01
description: "使用 Python 客户端连接 Easysearch，完成基础 CRUD 与搜索调用。"
summary: "Python 客户端 #  本页面帮助你快速跑通 Python 客户端连接 Easysearch 的完整流程。
推荐：Easysearch 官方 Python 客户端 #  Easysearch 提供了官方 Python 客户端 easysearch-py（Apache 2.0 开源），包名 easysearch。
 兼容说明：也可继续使用 elasticsearch-py 7.10.x 兼容连接，API 调用方式相同，仅导入路径不同。下文示例以官方客户端为主，兼容用法见 备选方案。
 安装依赖 #  # 官方客户端（推荐） pip install https://github.com/infinilabs/easysearch-py/releases/download/v0.1.0/easysearch-0.1.0-py2.py3-none-any.whl # 如需 async/await 支持 pip install &#34;easysearch[async] @ https://github.com/infinilabs/easysearch-py/releases/download/v0.1.0/easysearch-0.1.0-py2.py3-none-any.whl&#34; 建立连接 #  from easysearch import Easysearch es = Easysearch( [&#34;https://localhost:9200&#34;], http_auth=(&#34;admin&#34;, &#34;your_password&#34;), verify_certs=False, # 开发环境；生产环境请配置 CA 证书 timeout=30, max_retries=3, retry_on_timeout=True, ) # 验证连接 print(es."
---


# Python 客户端

本页面帮助你快速跑通 Python 客户端连接 Easysearch 的完整流程。

## 推荐：Easysearch 官方 Python 客户端

Easysearch 提供了官方 Python 客户端 [easysearch-py](https://github.com/infinilabs/easysearch-py)（Apache 2.0 开源），包名 `easysearch`。

> **兼容说明**：也可继续使用 `elasticsearch-py` 7.10.x 兼容连接，API 调用方式相同，仅导入路径不同。下文示例以官方客户端为主，兼容用法见[备选方案](#备选使用-elasticsearch-py-兼容连接)。

## 安装依赖

```bash
# 官方客户端（推荐）
pip install https://github.com/infinilabs/easysearch-py/releases/download/v0.1.0/easysearch-0.1.0-py2.py3-none-any.whl

# 如需 async/await 支持
pip install "easysearch[async] @ https://github.com/infinilabs/easysearch-py/releases/download/v0.1.0/easysearch-0.1.0-py2.py3-none-any.whl"
```

## 建立连接

```python
from easysearch import Easysearch

es = Easysearch(
    ["https://localhost:9200"],
    http_auth=("admin", "your_password"),
    verify_certs=False,       # 开发环境；生产环境请配置 CA 证书
    timeout=30,
    max_retries=3,
    retry_on_timeout=True,
)

# 验证连接
print(es.info())
```

### 生产环境（使用 CA 证书）

```python
from ssl import create_default_context

context = create_default_context(cafile="/path/to/root-ca.pem")
es = Easysearch(
    ["https://easysearch-host:9200"],
    ssl_context=context,
    http_auth=("admin", "your_password"),
    timeout=30,
)
```

## 索引文档

```python
doc = {
    "title": "Easysearch 入门",
    "content": "分布式搜索引擎快速上手",
    "tags": ["搜索", "教程"],
    "views": 100,
}

resp = es.index(index="articles", id=1, body=doc)
print(resp["result"])  # created
```

## 批量写入

```python
from easysearch.helpers import bulk

actions = [
    {"_index": "articles", "_id": i, "_source": {"title": f"文章 {i}", "views": i * 10}}
    for i in range(2, 102)
]

success, errors = bulk(es, actions)
print(f"成功写入 {success} 条")
```

## 获取文档

```python
resp = es.get(index="articles", id=1)
print(resp["_source"])
```

## 搜索

### 全文搜索

```python
resp = es.search(
    index="articles",
    body={
        "query": {"match": {"title": "Easysearch"}},
    },
)

for hit in resp["hits"]["hits"]:
    print(f'{hit["_score"]:.2f}  {hit["_source"]["title"]}')
```

### Bool 组合查询

```python
resp = es.search(
    index="articles",
    body={
        "query": {
            "bool": {
                "must": [{"match": {"title": "文章"}}],
                "filter": [{"range": {"views": {"gte": 500}}}],
            }
        },
        "sort": [{"views": "desc"}],
        "size": 5,
    },
)
```

## 聚合

```python
resp = es.search(
    index="articles",
    body={
        "size": 0,
        "aggs": {
            "tag_count": {"terms": {"field": "tags.keyword"}},
            "avg_views": {"avg": {"field": "views"}},
        },
    },
)

for bucket in resp["aggregations"]["tag_count"]["buckets"]:
    print(f'{bucket["key"]}: {bucket["doc_count"]}')
```

## 更新与删除

```python
# 局部更新
es.update(index="articles", id=1, body={"doc": {"views": 200}})

# 删除文档
es.delete(index="articles", id=1)

# 删除索引
es.indices.delete(index="articles")
```

## 注意事项

| 事项 | 说明 |
|------|------|
| 推荐客户端 | **`easysearch-py`**（官方客户端，包名 `easysearch`） |
| 兼容客户端 | `elasticsearch==7.10.1` 也可正常使用 |
| 证书 | 开发环境可 `verify_certs=False`；生产环境应配置 CA 证书 |
| 多节点 | 传入列表 `["https://node1:9200", "https://node2:9200"]` |

## 备选：使用 elasticsearch-py 兼容连接

Easysearch 兼容 Elasticsearch 7.10 API，也可以使用 `elasticsearch-py` 7.10.x 客户端：

```bash
pip install elasticsearch==7.10.1
```

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(
    ["https://localhost:9200"],
    http_auth=("admin", "your_password"),
    verify_certs=False,
    timeout=30,
)
```

> 兼容客户端的 API 调用方式完全相同，仅导入路径不同：`from elasticsearch import Elasticsearch`、`from elasticsearch.helpers import bulk`。

## 相关文档

- [easysearch-py GitHub](https://github.com/infinilabs/easysearch-py)
- [使用 Curl 访问 Easysearch]({{< relref "../../quick-start/connect/curl.md" >}})
- [Java 客户端]({{< relref "./java.md" >}})
- [入门教程]({{< relref "../../quick-start/tutorial.md" >}})

