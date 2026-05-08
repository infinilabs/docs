---
title: "Docker Compose"
date: 0001-01-01
summary: "Docker Compose 环境下使用 Easysearch #  在使用 docker-compose 运行 Easysearch 集群之前，请确保已进行 系统调优并安装好 Docker 服务，且 Docker 服务正常运行。
# 安装docker-compose curl -L &#34;https://github.com/docker/compose/releases/download/v2.6.1/docker-compose-$(uname -s)-$(uname -m)&#34; -o /usr/local/bin/docker-compose # 增加执行权限 chmod +x /usr/local/bin/docker-compose # 检查版本信息 docker-compose -v 运行 2 节点 docker compose 项目 #  从官网下载文件并解压，然后运行初始化脚本，最后运行启动脚本。
 在宿主机上创建工作目录  # 创建操作目录 sudo mkdir -p /data/docker/compose 下载文件并解压   如需测试 3 节点，只需把下面的下载文件名改为 3node.tar.gz 即可。
 curl -sSL https://release.infinilabs.com/easysearch/archive/compose/2node.tar.gz | sudo tar -xzC /data/docker/compose --strip-components=1 # 调整目录权限 sudo chown -R ${USER} /data/docker/compose  注意：解压之后，请把镜像的 latest 版本手工更新成具体的版本，可参考下面的命令"
---


# Docker Compose 环境下使用 Easysearch

在使用 docker-compose 运行 Easysearch 集群之前，请确保已进行[系统调优]({{< relref "/docs/deployment/config/settings" >}})并安装好 Docker 服务，且 Docker 服务正常运行。

```bash
# 安装docker-compose
curl -L "https://github.com/docker/compose/releases/download/v2.6.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 增加执行权限
chmod +x /usr/local/bin/docker-compose
# 检查版本信息
docker-compose -v
```

## 运行 2 节点 docker compose 项目

从官网下载文件并解压，然后运行初始化脚本，最后运行启动脚本。

1. 在宿主机上创建工作目录

```bash
# 创建操作目录
sudo mkdir -p /data/docker/compose
```

2. 下载文件并解压

> 如需测试 3 节点，只需把下面的下载文件名改为 [3node.tar.gz](https://release.infinilabs.com/easysearch/archive/compose/3node.tar.gz) 即可。

```bash
curl -sSL https://release.infinilabs.com/easysearch/archive/compose/2node.tar.gz | sudo tar -xzC /data/docker/compose --strip-components=1
# 调整目录权限
sudo chown -R ${USER} /data/docker/compose
```
> 注意：解压之后，请把镜像的 latest 版本手工更新成具体的版本，可参考下面的命令
>  ```bash
>  cd /data/docker/compose
> # 替换成具体的版本, 注：如果是 MacOS 请调整为 sudo sed -i '' "s/easysearch:latest/easysearch:{{< globaldata "easysearch" "version" >}}/;s/console:latest/console:{{< globaldata "console" "version" >}}/" docker-compose.yml
> sudo sed -i "s/easysearch:latest/easysearch:{{< globaldata "easysearch" "version" >}}/;s/console:latest/console:{{< globaldata "console" "version" >}}/" docker-compose.yml
> #检查版本是否替换成功
> grep image docker-compose.yml
>  ```

3. 运行 docker-compose 项目

```bash
cd /data/docker/compose
# 调整目录权限
sudo ./init.sh
# 下载镜像
docker-compose pull
# 启动 docker-compose 项目
./start.sh
# 如果启动失败，提示没权限，可进行权限调整，如 MacOS Docker 需要使用当前用户权限
sudo chown -R ${USER}:staff /data/docker/compose
```

4. 停止 docker-compose 项目

```bash
# 停止项目
./stop.sh
```

5. 清理 docker-compose 项目

```bash
# 清理项目，将清理 Easysearch 的 data 和 logs。
sudo ./reset.sh
```

6. 检查集群节点

```bash
# 集群默认的密码为admin
curl -ku admin:xxxxxxxxxxxx https://localhost:9201/_cat/nodes?v
# 输出信息如下：
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.24.0.3           68          31  31    1.67    0.57     0.21 dimr      -      easysearch-node1
172.24.0.2           55          31  31    1.67    0.57     0.21 dimr      *      easysearch-node2
```

## 未获取到最新镜像处理方法

```bash
# 通过访问 https://hub.docker.com/r/infinilabs/easysearch/tags ，然后点击 latest 找出镜像的 sha256 值。
HASH=803b8aab2ec012728901112e916f1aa0fadc85c9b6b21b887a051aa8c5e53e8a
docker pull infinilabs/easysearch:latest@sha256:$HASH
IMGID=$(docker image ls --format "table {{.ID}}" --digests |grep "$HASH" |awk '{print $1}')
docker tag $IMGID infinilabs/easysearch:latest
docker images |grep -v none |grep $IMGID

# 或者直接通过版本号进行镜像拉取
docker pull infinilabs/easysearch:{{< globaldata "easysearch" "version" >}}
```

