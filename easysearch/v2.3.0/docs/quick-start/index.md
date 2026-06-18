---
title: "快速开始"
date: 0001-01-01
description: "15 分钟快速起步，安装、连接与核心功能体验。"
summary: "快速开始 #  15 分钟快速上手 Easysearch，完成安装 → 连接 → 写入 → 查询的完整链路。
第一步：安装 #  选择适合你的方式快速安装，详见  如何安装。
# 最快方式：一键安装（Linux） curl -sSL http://get.infini.cloud | bash -s -- -p easysearch cd /data/easysearch &amp;&amp; bin/initialize.sh -s chown -R easysearch:easysearch /data/easysearch su easysearch -c &#34;/data/easysearch/bin/easysearch -d&#34; 第二步：验证与连接 #  初始化完成后，admin 密码会直接输出在终端中，请务必记住。也可以通过环境变量 EASYSEARCH_INITIAL_ADMIN_PASSWORD 预先指定密码。
# 验证服务（将 YOUR_PASSWORD 替换为初始化时终端输出的密码） curl -ku admin:YOUR_PASSWORD https://localhost:9200 更多访问方式见  如何使用：
  使用 Curl 访问：命令行方式  使用 Easysearch UI：内置 Web 界面，开箱即用  使用 INFINI Console：多集群统一管理平台  Java 客户端：Java 应用集成  Python 客户端：Python 应用集成  第三步：动手实践 #  通过  入门教程 学习核心操作："
---


# 快速开始

15 分钟快速上手 Easysearch，完成安装 → 连接 → 写入 → 查询的完整链路。

## 第一步：安装

选择适合你的方式快速安装，详见 **[如何安装]({{< relref "./install/" >}})**。

```bash
# 最快方式：一键安装（Linux）
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch
cd /data/easysearch && bin/initialize.sh -s
chown -R easysearch:easysearch /data/easysearch
su easysearch -c "/data/easysearch/bin/easysearch -d"
```

## 第二步：验证与连接

初始化完成后，admin 密码会直接输出在终端中，请务必记住。也可以通过环境变量 `EASYSEARCH_INITIAL_ADMIN_PASSWORD` 预先指定密码。

```bash
# 验证服务（将 YOUR_PASSWORD 替换为初始化时终端输出的密码）
curl -ku admin:YOUR_PASSWORD https://localhost:9200
```

更多访问方式见 **[如何使用]({{< relref "./connect/" >}})**：

- [使用 Curl 访问]({{< relref "./connect/curl.md" >}})：命令行方式
- [使用 Easysearch UI]({{< relref "./connect/easysearch-ui.md" >}})：内置 Web 界面，开箱即用
- [使用 INFINI Console]({{< relref "./connect/console.md" >}})：多集群统一管理平台
- [Java 客户端]({{< relref "./connect/java-client.md" >}})：Java 应用集成
- [Python 客户端]({{< relref "./connect/python-client.md" >}})：Python 应用集成

## 第三步：动手实践

通过 **[入门教程]({{< relref "./tutorial.md" >}})** 学习核心操作：

1. 创建索引与写入文档
2. 全文搜索与过滤
3. 聚合分析
4. 高亮与短语搜索

## 5 分钟速览

如果你现在就想快速体验核心功能：

```bash
# 1. 写入文档
curl -ku admin:YOUR_PASSWORD -X POST "https://localhost:9200/demo/_doc/1" \
  -H 'Content-Type: application/json' \
  -d '{"title": "Easysearch 入门", "tags": ["搜索", "数据库"], "views": 100}'

curl -ku admin:YOUR_PASSWORD -X POST "https://localhost:9200/demo/_doc/2" \
  -H 'Content-Type: application/json' \
  -d '{"title": "分布式搜索原理", "tags": ["搜索", "分布式"], "views": 250}'

# 2. 全文搜索
curl -ku admin:YOUR_PASSWORD "https://localhost:9200/demo/_search?q=搜索"

# 3. 结构化查询
curl -ku admin:YOUR_PASSWORD -X POST "https://localhost:9200/demo/_search" \
  -H 'Content-Type: application/json' \
  -d '{
    "query": {
      "bool": {
        "must": [{"match": {"title": "搜索"}}],
        "filter": [{"range": {"views": {"gte": 200}}}]
      }
    }
  }'

# 4. 聚合统计
curl -ku admin:YOUR_PASSWORD -X POST "https://localhost:9200/demo/_search" \
  -H 'Content-Type: application/json' \
  -d '{
    "size": 0,
    "aggs": {"tag_count": {"terms": {"field": "tags.keyword"}}}
  }'
```

## 下一步

完成快速开始后，建议继续学习：

| 方向 | 推荐章节 |
|------|----------|
| 理解原理 | [基础理论]({{< relref "../fundamentals/" >}})：倒排索引、分布式、评分 |
| 生产部署 | [部署手册]({{< relref "../deployment/" >}})：节点规划、系统调优、云平台部署 |
| 深入功能 | [功能手册]({{< relref "../features/" >}})：Query DSL、聚合、Mapping、SQL |
| AI 搜索 | [AI 集成]({{< relref "../integrations/ai/" >}})：向量检索、RAG、LLM |
