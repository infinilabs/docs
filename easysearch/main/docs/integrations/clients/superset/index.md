---
title: "Superset 集成"
date: 0001-01-01
description: "将 Easysearch 作为数据源接入 Apache Superset，构建可视化报表与数据看板。"
summary: "Apache Superset 集成 #  Apache Superset 是一款强大的开源数据可视化与 BI 平台。通过 Elasticsearch 连接器，Superset 可以直接查询 Easysearch 中的数据并构建交互式看板。
相关指南 #    SQL 查询接口  Grafana 集成  前置条件 #     条件 说明     Superset 版本 2.0+ 推荐   Python 驱动 elasticsearch-dbapi 包   网络可达 Superset 服务器能够访问 Easysearch 端口   Easysearch SQL 功能 确保 SQL 插件已启用    安装驱动 #  在 Superset 运行环境中安装 Elasticsearch 驱动："
---


# Apache Superset 集成

Apache Superset 是一款强大的开源数据可视化与 BI 平台。通过 Elasticsearch 连接器，Superset 可以直接查询 Easysearch 中的数据并构建交互式看板。

## 相关指南

- [SQL 查询接口]({{< relref "./sql.md" >}})
- [Grafana 集成]({{< relref "../third-party/grafana.md" >}})

## 前置条件

| 条件                   | 说明                                        |
| ---------------------- | ------------------------------------------- |
| Superset 版本          | 2.0+ 推荐                                   |
| Python 驱动            | `elasticsearch-dbapi` 包                     |
| 网络可达               | Superset 服务器能够访问 Easysearch 端口      |
| Easysearch SQL 功能    | 确保 SQL 插件已启用                          |

## 安装驱动

在 Superset 运行环境中安装 Elasticsearch 驱动：

```bash
pip install elasticsearch-dbapi
```

如果使用 Docker 部署的 Superset，需要在自定义镜像中添加此依赖：

```dockerfile
FROM apache/superset:latest
RUN pip install elasticsearch-dbapi
```

## 配置数据源

### 1. 添加数据库连接

在 Superset 中进入 **Settings → Database Connections → + Database**，选择 Elasticsearch。

连接字符串格式：

```
elasticsearch+https://admin:password@easysearch-host:9200
```

如果使用 HTTP（开发环境）：

```
elasticsearch+http://admin:password@easysearch-host:9200
```

### 2. 高级配置

在 **Advanced** → **Other** → **Engine Parameters** 中可以设置额外参数：

```json
{
  "connect_args": {
    "verify_certs": false,
    "timeout": 60
  }
}
```

| 参数             | 说明                     | 默认值 |
| ---------------- | ------------------------ | ------ |
| `verify_certs`   | 是否验证 SSL 证书        | true   |
| `timeout`        | 查询超时（秒）           | 30     |
| `http_compress`  | 是否启用 HTTP 压缩       | false  |

### 3. 测试连接

点击 **Test Connection** 确认连接成功。

## 创建数据集与图表

### 添加数据集

1. 进入 **Datasets → + Dataset**
2. 选择刚配置的 Easysearch 数据库
3. Schema 留空，Table 选择要分析的索引名
4. 保存数据集

### 适合的图表类型

| 图表类型     | 适用场景                 | 对应查询能力                  |
| ------------ | ------------------------ | ----------------------------- |
| 时间序列折线 | 日志趋势、指标监控       | `date_histogram` 聚合         |
| 柱状图/饼图  | 分类分布统计             | `terms` 聚合                  |
| 数字卡片     | KPI 指标展示             | `sum` / `avg` / `count` 聚合  |
| 表格         | 明细数据查看             | `SELECT` 查询                 |
| 热力图       | 时间 × 维度交叉分析      | 多级聚合                      |

## 性能优化建议

- **索引设计**：为 Superset 常用查询场景预创建聚合友好的索引映射
- **查询超时**：对大索引设置合理的 `timeout` 参数，避免长时间查询阻塞
- **缓存策略**：在 Superset 中为慢查询的数据集启用缓存，减少对 Easysearch 的重复请求
- **采样查询**：在探索阶段使用 `LIMIT` 限制返回行数
- **专用账户**：为 Superset 创建只读的专用账户，限制其可访问的索引范围


