---
title: "测试环境部署"
date: 0001-01-01
summary: "测试环境部署指南 #  用最少资源快速搭建一个可运行的 Easysearch 实例，适合功能验证、开发联调和学习使用。
最低硬件要求 #     项目 最低配置 建议配置     CPU 2 核 4 核   内存 4 GB 8 GB   磁盘 20 GB 50 GB SSD   JDK 11+ 17+     测试环境可使用 HDD，但 SSD 体验更佳。
 一键安装（推荐） #  # 使用一键安装脚本 curl -sSL http://get.infini.cloud | bash -s -- -p easysearch # 调小 JVM 堆（测试环境 512MB 即可） sed -i &#39;s/1g/512m/g&#39; /data/easysearch/config/jvm."
---


# 测试环境部署指南

用最少资源快速搭建一个可运行的 Easysearch 实例，适合功能验证、开发联调和学习使用。

## 最低硬件要求

| 项目 | 最低配置 | 建议配置 |
|------|:--------:|:--------:|
| CPU | 2 核 | 4 核 |
| 内存 | 4 GB | 8 GB |
| 磁盘 | 20 GB | 50 GB SSD |
| JDK | 11+ | 17+ |

> 测试环境可使用 HDD，但 SSD 体验更佳。

## 一键安装（推荐）

```bash
# 使用一键安装脚本
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch

# 调小 JVM 堆（测试环境 512MB 即可）
sed -i 's/1g/512m/g' /data/easysearch/config/jvm.options

# 初始化
cd /data/easysearch && bin/initialize.sh -s

# 调整目录权限
chown -R easysearch:easysearch /data/easysearch

# 以 easysearch 用户启动
su easysearch -c "/data/easysearch/bin/easysearch -d"
```

初始化完成后，admin 密码会直接输出在终端中，请务必记住。也可以通过环境变量 `EASYSEARCH_INITIAL_ADMIN_PASSWORD` 预先指定密码（需要 1.8.2 及以后版本）。

## Docker 快速启动

如果已安装 Docker，可以用一条命令启动：

```bash
docker run -d \
  --name easysearch-test \
  -p 9200:9200 \
  -e "EASYSEARCH_INITIAL_ADMIN_PASSWORD=MyTest@2024" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  infinilabs/easysearch:latest
```

> 更多 Docker 选项见 [Docker 部署]({{< relref "./docker.md" >}})。

## MacOS 快速体验

```bash
# Homebrew 安装 JDK
brew install openjdk@17

# 下载并解压 Easysearch
# 具体下载地址见官方页面

cd easysearch
bin/initialize.sh -s
bin/easysearch
```

> 详见 [MacOS 部署]({{< relref "./macos.md" >}})。

## 验证安装

```bash
# 测试连接（将 YOUR_PASSWORD 替换为初始化时终端输出的密码）
curl -ku admin:YOUR_PASSWORD https://localhost:9200

# 预期输出（示例）
{
  "name": "node-1",
  "cluster_name": "easysearch",
  "cluster_uuid": "...",
  "version": {
    "distribution": "easysearch",
    "number": "x.y.z",
    ...
  },
  "tagline": "You Know, For Easy Search!"
}
```

## 快速写入与查询

```bash
# 写入一条文档
curl -ku admin:YOUR_PASSWORD -X POST "https://localhost:9200/test/_doc/1" \
  -H 'Content-Type: application/json' \
  -d '{"title": "Hello Easysearch", "content": "测试环境验证成功"}'

# 搜索
curl -ku admin:YOUR_PASSWORD "https://localhost:9200/test/_search?q=Hello"
```

## 注意事项

- 测试环境**不建议**用于存放重要数据
- 单节点无高可用保障，集群状态为 `yellow`（因为副本无法分配到其他节点）
- 如需模拟分布式集群，可使用 [Docker Compose]({{< relref "./docker-compose.md" >}}) 或 [分布式集群部署]({{< relref "./cluster.md" >}})

## 延伸阅读

- [生产环境部署]({{< relref "./production-env.md" >}})
- [Docker Compose 集群]({{< relref "./docker-compose.md" >}})
- [入门教程]({{< relref "/docs/quick-start/tutorial.md" >}})

