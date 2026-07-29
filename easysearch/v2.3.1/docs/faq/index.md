---
title: "FAQ"
date: 0001-01-01
description: "常见问题与解答。"
summary: "常见问题 #  Easysearch 使用过程中的高频问题与快速解答。
产品与兼容性 #  Easysearch 和 Elasticsearch 是什么关系？ #  Easysearch 兼容 ES 7.10 API 与生态工具，原有基于 ES 7.x 构建的应用和工作流可以平滑迁移。在此基础上，Easysearch 持续迭代，新增了向量检索、内置安全、国产化适配等企业级增强功能，并采用商用友好协议。
Easysearch 兼容哪些 ES 客户端和工具？ #   客户端：ES 7.10.2 OSS 版本的 Java/Python/Go 等客户端均可直接使用 工具：Logstash 7.10.2 OSS、Filebeat 7.10.2 OSS、Kibana 7.10.2 OSS 可直接连接 ORM 框架：Easy-ES、Spring Data Elasticsearch（2.x 系列）均兼容  需要在 easysearch.yml 中开启兼容模式：
elasticsearch.api_compatibility: true Easysearch 是否开源？ #  Easysearch 不是开源软件，是一款商业软件，采用商用友好协议。
商用友好协议的核心优势：
 无限度的内部使用自由：企业可在任何规模的生产环境中自由部署和使用 完整的二次开发权：允许深度定制和功能扩展，衍生成果完全归企业所有 无限制的商业化权利：可基于 Easysearch 构建 SaaS 服务、托管服务或商业产品，无需公开源代码 无传染性风险：不会因为提供服务而被迫公开技术栈  安装与部署 #  最低系统要求是什么？ #     项目 最低 推荐（生产）     CPU 2 核 16 核+   内存 4 GB 64 GB+   JDK 11 17+   磁盘 20 GB SSD，按数据量规划     详见 测试环境部署 和 生产环境部署。"
---


# 常见问题

Easysearch 使用过程中的高频问题与快速解答。

## 产品与兼容性

### Easysearch 和 Elasticsearch 是什么关系？

Easysearch 兼容 ES 7.10 API 与生态工具，原有基于 ES 7.x 构建的应用和工作流可以平滑迁移。在此基础上，Easysearch 持续迭代，新增了向量检索、内置安全、国产化适配等企业级增强功能，并采用商用友好协议。

### Easysearch 兼容哪些 ES 客户端和工具？

- **客户端**：ES 7.10.2 OSS 版本的 Java/Python/Go 等客户端均可直接使用
- **工具**：Logstash 7.10.2 OSS、Filebeat 7.10.2 OSS、Kibana 7.10.2 OSS 可直接连接
- **ORM 框架**：Easy-ES、Spring Data Elasticsearch（2.x 系列）均兼容

需要在 `easysearch.yml` 中开启兼容模式：

```yaml
elasticsearch.api_compatibility: true
```

### Easysearch 是否开源？

Easysearch 不是开源软件，是一款商业软件，采用商用友好协议。

商用友好协议的核心优势：

- **无限度的内部使用自由**：企业可在任何规模的生产环境中自由部署和使用
- **完整的二次开发权**：允许深度定制和功能扩展，衍生成果完全归企业所有
- **无限制的商业化权利**：可基于 Easysearch 构建 SaaS 服务、托管服务或商业产品，无需公开源代码
- **无传染性风险**：不会因为提供服务而被迫公开技术栈

## 安装与部署

### 最低系统要求是什么？

| 项目 | 最低 | 推荐（生产） |
|------|------|-------------|
| CPU | 2 核 | 16 核+ |
| 内存 | 4 GB | 64 GB+ |
| JDK | 11 | 17+ |
| 磁盘 | 20 GB | SSD，按数据量规划 |

> 详见 [测试环境部署]({{< relref "../deployment/install-guide/test-env.md" >}}) 和 [生产环境部署]({{< relref "../deployment/install-guide/production-env.md" >}})。

### 安装后忘记了 admin 密码怎么办？

初始密码仅在执行 `bin/initialize.sh` 时的终端输出中显示一次，不会保存到文件。如果丢失，可通过内置证书重置：

```bash
# 使用 security 工具重置密码
bin/security-admin.sh -cd config/security -cacert config/root-ca.pem \
  -cert config/admin.pem -key config/admin-key.pem -nhnv
```

> 参考博客：[Easysearch 集群重置 admin 用户密码](https://infinilabs.cn/blog/2025/easysearch-reset-admin-password/)

### Docker 部署时提示 `max virtual memory areas vm.max_map_count [65530] is too low` 怎么办？

```bash
# 宿主机执行
sudo sysctl -w vm.max_map_count=262144
# 持久化
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

### 支持哪些操作系统？

- **Linux**：CentOS 7+、Ubuntu 18.04+、Debian 10+ 及各信创平台（麒麟、统信、龙芯、鲲鹏等）
- **Windows**：支持，但不建议用于生产
- **macOS**：支持，适合开发测试
- **容器化**：Docker、Docker Compose、Kubernetes (Helm/Operator)

> 信创平台参考 [信创安装指南]({{< relref "../deployment/install-guide/ciip/" >}})。

## 连接与客户端

### 连接时报 SSL/TLS 证书错误？

Easysearch 默认使用自签名证书。客户端连接时需要：

- **curl**：使用 `-k` 跳过验证，或 `--cacert root-ca.pem` 指定 CA
- **Java**：配置 SSLContext 信任自签名证书
- **Python**：设置 `verify_certs=False` 或指定 CA 路径

```bash
# curl 示例
curl -ku admin:password https://localhost:9200
```

### 如何使用 Logstash / Filebeat 连接？

使用 **7.10.2 OSS** 版本的 Logstash / Filebeat，output 配置示例：

```yaml
# Logstash
output {
  elasticsearch {
    hosts => ["https://localhost:9200"]
    user => "admin"
    password => "your-password"
    ssl => true
    ssl_certificate_verification => false
    ilm_enabled => false
  }
}
```

```yaml
# Filebeat
output.elasticsearch:
  hosts: ["https://localhost:9200"]
  username: "admin"
  password: "your-password"
  ssl.verification_mode: none
```

## 索引与查询

### 集群状态显示 yellow 是什么意思？

`yellow` 表示所有主分片正常，但有副本分片未分配。常见于：

- **单节点集群**：副本无法分配到其他节点（正常现象）
- **节点故障**：某个节点下线导致副本丢失

单节点可以将副本数设为 0：

```
PUT /my-index/_settings
{
  "number_of_replicas": 0
}
```

### 中文搜索效果不好怎么办？

默认的 `standard` 分析器按字符切分中文，效果差。需要使用 IK 分词器：

```json
PUT /my-index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

> 详见 [文本分析]({{< relref "../features/mapping-and-analysis/text-analysis/" >}}) 和 [NLP]({{< relref "../fundamentals/nlp.md" >}})。

### 写入速度慢怎么优化？

1. 使用 `_bulk` API 批量写入（每批 5~15 MB）
2. 增大 `refresh_interval`（如 `30s`）减少段创建频率
3. 写入期间临时设置 `number_of_replicas: 0`
4. 调整 `index.translog.durability: async` 和 `index.translog.flush_threshold_size: 512mb`

> 详见 [Bulk API]({{< relref "../features/document-operations/bulk-api.md" >}})。

### 深度分页（from + size 超过 10000）报错？

默认 `index.max_result_window` 为 10000。深度分页建议使用 `search_after`：

```json
GET /my-index/_search
{
  "size": 10,
  "sort": [{ "timestamp": "desc" }, { "_id": "asc" }],
  "search_after": ["2024-01-01T00:00:00Z", "doc_12345"]
}
```

> 详见 [分页与排序]({{< relref "../features/fulltext-search/pagination-and-sorting.md" >}})。

## 运维与监控

### 如何备份和恢复数据？

使用快照 API：

```bash
# 注册仓库
PUT _snapshot/my_backup
{ "type": "fs", "settings": { "location": "/mnt/backup" } }

# 创建快照
PUT _snapshot/my_backup/snap_1

# 恢复
POST _snapshot/my_backup/snap_1/_restore
```

> 详见 [备份与恢复]({{< relref "/docs/features/data-retention/backup-restore" >}})。

### 磁盘满了怎么办？

1. 删除不需要的旧索引：`DELETE /old-index-*`
2. 清理已关闭的索引
3. 如果集群进入只读模式，手动解除：

```
PUT _all/_settings
{
  "index.blocks.read_only_allow_delete": null
}
```

4. 长期方案：配置 ILM 自动清理过期数据

> 详见 [数据生命周期]({{< relref "/docs/features/data-retention/lifecycle" >}}) 和 [故障排查]({{< relref "../operations/troubleshooting.md" >}})。

## 更多资源

- [K8s Operator FAQ]({{< relref "../deployment/install-guide/operator/FAQ.md" >}})
- [故障排查]({{< relref "../operations/troubleshooting.md" >}})
- [其它资源（博客、视频）]({{< relref "../resources/" >}})
