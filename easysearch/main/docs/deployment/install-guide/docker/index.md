---
title: "Docker"
date: 0001-01-01
summary: "Docker 环境下使用 Easysearch #  在使用 Docker 运行 Easysearch 之前，请确保已进行 系统调优并安装好 Docker 服务，且 Docker 服务正常运行。
 最快方式：启动临时的 docker 容器，可以从前台查看到 admin 随机生成的初始密码
 注： Docker 环境一般用于临时验证，如需要长期使用请务必进行数据持久化   # 直接运行镜像使用随机密码（数据及配置未持久化） docker run --name easysearch --ulimit memlock=-1:-1 -p 9200:9200 infinilabs/easysearch:2.1.2-2696 # 使用自定义密码，可以使用环境变量配置 （需要 1.8.2 及以后的版本才支持） echo &#34;EASYSEARCH_INITIAL_ADMIN_PASSWORD=$(openssl rand -hex 10)&#34; | tee .env # 通过从环境变量文件设置初始密码（数据及配置未持久化） docker run --name easysearch --env-file ./.env --ulimit memlock=-1:-1 -p 9200:9200 infinilabs/easysearch:2.1.2-2696 # 使用自定义密码及命名卷 (数据持久化到命名卷) docker run -d --name easysearch \  --ulimit memlock=-1:-1 \  --env-file ."
---


# Docker 环境下使用 Easysearch

在使用 Docker 运行 Easysearch 之前，请确保已进行[系统调优]({{< relref "/docs/deployment/config/settings" >}})并安装好 Docker 服务，且 Docker 服务正常运行。

> 最快方式：启动临时的 docker 容器，可以从前台查看到 admin 随机生成的初始密码
> - 注： Docker 环境一般用于临时验证，如需要长期使用请务必进行**数据持久化**

```bash
# 直接运行镜像使用随机密码（数据及配置未持久化）
docker run --name easysearch --ulimit memlock=-1:-1 -p 9200:9200 infinilabs/easysearch:{{< globaldata "easysearch" "version" >}}

# 使用自定义密码，可以使用环境变量配置 （需要 1.8.2 及以后的版本才支持）
echo "EASYSEARCH_INITIAL_ADMIN_PASSWORD=$(openssl rand -hex 10)" | tee .env

# 通过从环境变量文件设置初始密码（数据及配置未持久化）
docker run --name easysearch --env-file ./.env --ulimit memlock=-1:-1 -p 9200:9200 infinilabs/easysearch:{{< globaldata "easysearch" "version" >}}

# 使用自定义密码及命名卷 (数据持久化到命名卷)
docker run -d --name easysearch \
  --ulimit memlock=-1:-1 \
  --env-file ./.env -p 9200:9200 \
  -v easysearch-data:/app/easysearch/data \
  -v easysearch-config:/app/easysearch/config \
  -v easysearch-logs:/app/easysearch/logs \
  infinilabs/easysearch:{{< globaldata "easysearch" "version" >}}
```

## 数据持久化到本地（数据可长期使用）

设置自定义密码，并从宿主机挂载数据目录、配置目录及日志目录，配置 jvm 内存为 512m。

1. 在宿主机上创建目录

```bash
# 创建数据及日志存储目录
sudo mkdir -p /data/easysearch/{data,logs}
# 根据自己的需求，设置成安全的密码
echo "EASYSEARCH_INITIAL_ADMIN_PASSWORD=$(openssl rand -hex 10)" | sudo tee /data/easysearch/.env
```

2. 修改目录权限

> **非必须步骤，视具体操作环境而定，可参考文末的错误说明**

```bash
# 注意：需要根据 Docker 运行环境判断是否需要调整权限，如在 MacOS 上使用 OrbStack 则不需要调整权限。
# 容器内 easysearch 用户的 uid 为 602，通过调整宿主机的目录权限，确保在容器内部 easysearch 用户有权限读写挂载的数据卷
sudo chown -R 602:602 /data/easysearch
```

3. 从镜像初始化 config 目录

```bash
# 将镜像中的 config 目录复制到本地目录
docker run --rm --env-file /data/easysearch/.env -v /data/easysearch:/work infinilabs/easysearch:{{< globaldata "easysearch" "version" >}} cp -rf /app/easysearch/config /work
```

4. 后台运行容器

```bash
#后台启动容器，并指定内存大小及挂载数据、日志目录，设定好容器名称及容器主机名称
#如需调整配置文件，可以修改以下配置文件
# 1. /data/easysearch/config/easysearch.yml
# 2. /data/easysearch/config/jvm.options
# 3. /data/easysearch/config/log4j2.properties
docker run -d --restart always -p 9200:9200 \
              --ulimit memlock=-1:-1 \
              -e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
              -v /data/easysearch/data:/app/easysearch/data \
              -v /data/easysearch/config:/app/easysearch/config \
              -v /data/easysearch/logs:/app/easysearch/logs \
              --name easysearch --hostname easysearch \
              infinilabs/easysearch:{{< globaldata "easysearch" "version" >}}
```

5. 升级 Easysearch 版本

```bash
# 先停止并删除正在运行的容器
docker stop easysearch && docker rm easysearch
# 再修改第 4 步中镜像的版本，重新运行命令即可
```

## 验证安装

```bash
# 查看容器状态
docker ps --filter name=easysearch

# 查看容器日志（首次运行时可从日志中找到 admin 随机密码）
docker logs easysearch

# 测试连接
curl -ku admin:YOUR_PASSWORD https://localhost:9200
```

## 健康检查

Docker 运行时可以配置健康检查，自动监测 Easysearch 是否正常运行：

```bash
docker run -d --name easysearch \
  --ulimit memlock=-1:-1 \
  --env-file ./.env -p 9200:9200 \
  -e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
  --health-cmd='curl -sku admin:${EASYSEARCH_INITIAL_ADMIN_PASSWORD} https://localhost:9200/_cluster/health || exit 1' \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=60s \
  -v easysearch-data:/app/easysearch/data \
  -v easysearch-config:/app/easysearch/config \
  -v easysearch-logs:/app/easysearch/logs \
  infinilabs/easysearch:{{< globaldata "easysearch" "version" >}}
```

查看健康状态：

```bash
docker inspect --format='{{.State.Health.Status}}' easysearch
```

## 常见问题

### vm.max_map_count 错误

```bash
# 临时生效
sudo sysctl -w vm.max_map_count=262144

# 永久生效
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

> 更多系统参数请参考 [系统调优]({{< relref "/docs/deployment/config/settings" >}})。

### 容器启动后立即退出

```bash
# 查看退出原因
docker logs easysearch

# 常见原因：
# 1. 内存不足 — 调整 ES_JAVA_OPTS
# 2. 权限问题 — 调整挂载目录权限为 chown 602:602 /data/easysearch
# 3. 端口冲突 — 更换映射端口
# 4. 容器名称冲突 - 删除异常容器 docker rm easysearch
```

#### 具体错误示例

- 权限错误

```bash
# 给目录赋予容器内 easysearch 用户操作权限，用户组和用户 ID 为 602
chown 602:602 /data/easysearch
```
{{% load-img "/errors/permission.svg" %}}


- 站口占用错误

```bash
# 修改端口映射，映射关系为 主机端口:容器端口
```
{{% load-img "/errors/port-already-in-use.svg" %}}

- 首次安装后重置密码

```bash
# 由于随机密码被日志覆盖，找不到或想重新设置密码可使用以下命令来重置
docker exec -it easysearch bash -c "/app/easysearch/bin/reset_admin_password.sh"
```
{{% load-img "/errors/reset-passwd.svg" %}}

### 查看容器内文件

```bash
# 进入容器
docker exec -it easysearch bash

# 容器内 Easysearch 路径
ls /app/easysearch/
```

## 延伸阅读

- [Docker Compose 集群]({{< relref "./docker-compose.md" >}})
- [测试环境部署]({{< relref "./test-env.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings" >}})
