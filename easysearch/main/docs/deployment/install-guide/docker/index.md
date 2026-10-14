---
title: "Docker"
date: 0001-01-01
summary: "Docker 环境下使用 Easysearch #  在使用 Docker 运行 Easysearch 之前，请确保已进行系统调优并安装好 Docker 服务，且 Docker 服务正常运行。
 最快方式：启动临时的 docker 容器，可以从前台查看到 admin 随机生成的初始密码
 注： Docker 环境一般用于临时验证，如需要长期使用请务必进行数据持久化   # 直接运行镜像使用随机密码（数据及配置未持久化） docker run --name easysearch --ulimit memlock=-1:-1 -p 9200:9200 infinilabs/easysearch:2.4.0-2975 # 使用自定义密码，可以使用环境变量配置 （需要 1.8.2 及以后的版本才支持） echo &#34;EASYSEARCH_INITIAL_ADMIN_PASSWORD=$(printf &#34;%s%s%s@%s&#34; &#34;$(tr -dc a-z&lt;/dev/urandom|head -c4)&#34; &#34;$(tr -dc A-Z&lt;/dev/urandom|head -c3)&#34; &#34;$(tr -dc 0-9&lt;/dev/urandom|head -c2)&#34; &#34;$(tr -dc a-z0-9&lt;/dev/urandom|head -c3)&#34;)&#34; | tee .env # 通过从环境变量文件设置初始密码（数据及配置未持久化） docker run --name easysearch --env-file ."
---


# Docker 环境下使用 Easysearch

在使用 Docker 运行 Easysearch 之前，请确保已进行[系统调优]({{< relref "/docs/deployment/config/settings" >}})并安装好 Docker 服务，且 Docker 服务正常运行。

> 最快方式：启动临时的 docker 容器，可以从前台查看到 admin 随机生成的初始密码
> - 注： Docker 环境一般用于临时验证，如需要长期使用请务必进行**数据持久化**

```bash
# 直接运行镜像使用随机密码（数据及配置未持久化）
docker run --name easysearch --ulimit memlock=-1:-1 -p 9200:9200 infinilabs/easysearch:{{< data "quick_start.easysearch.version" >}}

# 使用自定义密码，可以使用环境变量配置 （需要 1.8.2 及以后的版本才支持）
echo "EASYSEARCH_INITIAL_ADMIN_PASSWORD=$(printf "%s%s%s@%s" "$(tr -dc a-z</dev/urandom|head -c4)" "$(tr -dc A-Z</dev/urandom|head -c3)" "$(tr -dc 0-9</dev/urandom|head -c2)" "$(tr -dc a-z0-9</dev/urandom|head -c3)")" | tee .env

# 通过从环境变量文件设置初始密码（数据及配置未持久化）
docker run --name easysearch --env-file ./.env --ulimit memlock=-1:-1 -p 9200:9200 infinilabs/easysearch:{{< data "quick_start.easysearch.version" >}}

# 使用自定义密码及命名卷 (数据持久化到命名卷)
docker run -d --name easysearch \
  --ulimit memlock=-1:-1 \
  --env-file ./.env -p 9200:9200 \
  -v easysearch-data:/app/easysearch/data \
  -v easysearch-config:/app/easysearch/config \
  -v easysearch-logs:/app/easysearch/logs \
  infinilabs/easysearch:{{< data "quick_start.easysearch.version" >}}
```

## 数据持久化到本地（数据可长期使用）

设置自定义密码，并从宿主机挂载数据目录、配置目录及日志目录，配置 jvm 内存为 512m。

1. 在宿主机上创建目录

```bash
# 创建数据及日志存储目录
sudo mkdir -p /data/easysearch/{data,logs}
# 根据自己的需求，设置成安全的密码
echo "EASYSEARCH_INITIAL_ADMIN_PASSWORD=$(printf "%s%s%s@%s" "$(tr -dc a-z</dev/urandom|head -c4)" "$(tr -dc A-Z</dev/urandom|head -c3)" "$(tr -dc 0-9</dev/urandom|head -c2)" "$(tr -dc a-z0-9</dev/urandom|head -c3)")" | sudo tee /data/easysearch/.env
```

2. 修改目录权限

> **非必须步骤，视具体操作环境而定，可参考文末的错误说明**

```bash
# 注意：需要根据 Docker 运行环境判断是否需要调整权限，如在 MacOS 上使用 OrbStack 则不需要调整权限。
# 容器内 easysearch 用户的 uid 为 602，通过调整宿主机的目录权限，确保在容器内部 easysearch 用户有权限读写挂载的数据卷
sudo chown -R 602:602 /data/easysearch
```

> **不要用 `--user` 指定运行用户。** 容器应保持默认（root）启动：entrypoint 会先把
> `/app/easysearch/{data,config,logs}` 的属主修正为 `602:602`，再用 `gosu` 降权到 `easysearch`
> （uid 602）启动 Java 进程。若以 `--user 602:602` 启动，`gosu` 无法降权，容器会报
> `failed switching to "easysearch": operation not permitted` 后立即退出。
> 因此上面的 `chown 602:602` 只是为保证数据目录可写（在容器无法自行 chown 的环境，如 rootless
> Docker 或开启了 root squash 的 NFS 上才有必要），运行用户本身不需要、也不能改成 602。

3. 从镜像初始化 config 目录

> **必需步骤，不能省略。** 绑定挂载不像命名卷那样会自动填充镜像内容。如果挂载的 `config` 目录为空，
> 镜像自带的 `config/security/` 会被整体遮住，容器初始化时找不到
> `/app/easysearch/config/security/user.yml`，报 `ERROR: Admin password generation failed` 并立即退出
> （`docker logs` 中可见 `No such file or directory`）。
> 本步骤同时会把证书等初始化产物一并导出，复制出的文件属主为容器内 `easysearch` 用户（uid 602）。

```bash
# 将镜像中的 config 目录复制到本地目录
docker run --rm --env-file /data/easysearch/.env -v /data/easysearch:/work infinilabs/easysearch:{{< data "quick_start.easysearch.version" >}} cp -rf /app/easysearch/config /work
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
              infinilabs/easysearch:{{< data "quick_start.easysearch.version" >}}
```

5. 升级 Easysearch 版本

> **升级时必须同步版本相关的配置文件。** 镜像内置的 JDK 与 `config/jvm.options` 是按版本绑定的，
> 而 `config` 目录被持久化，旧版本留下的 `jvm.options` 会让新版本容器起不来。
> 以 `2.4.0`（内置 JDK 21）升到 `2.5.0`（内置 JDK 25）为例：
> - 旧配置里的 `-Djava.security.manager=allow` 在 JDK 25 上直接报
>   `A command line option has attempted to allow or enable the Security Manager`，VM 初始化失败；
> - 新版新增的 `-javaagent:lib/easysearch-security-agent-<版本>.jar` 在旧配置里不存在，
>   因此**只改前一个参数也不够**，会接着报 `java.lang.IllegalStateException: security agent is not loaded`。
>
> 结论：**不要只改单个参数，直接用新版本镜像同步整份 `jvm.options`**。

```bash
# 1. 先停止并删除正在运行的容器
docker stop easysearch && docker rm easysearch

# 2. 对比新旧 jvm.options 是否一致（容器内没有 diff/cmp，用 sha256sum 判断；哈希不同即需要同步）
docker run --rm -v /data/easysearch/config:/c infinilabs/easysearch:<新版本> \
  sha256sum /app/easysearch/config/jvm.options /c/jvm.options

# 3. 用新版本镜像同步版本相关的配置
#    该命令以容器内 easysearch 用户执行，属主保持 602，无需再 chown
docker run --rm -v /data/easysearch/config:/c infinilabs/easysearch:<新版本> \
  cp -f /app/easysearch/config/jvm.options /c/jvm.options

# 4. 按第 4 步、使用新的镜像版本重新运行容器
```

> 需要查看具体差异时，先把新版本的 `jvm.options` 复制出来，再在宿主机上比对（容器内没有 `diff`）：
> ```bash
> docker run --rm -v /data/easysearch:/work infinilabs/easysearch:<新版本> \
>   cp -f /app/easysearch/config/jvm.options /work/jvm.options.new
> diff /data/easysearch/config/jvm.options /data/easysearch/jvm.options.new
> ```
>
> 用命名卷存 `config` 时（见文首"使用自定义密码及命名卷"）同样需要同步，把上面的
> `-v /data/easysearch/config:/c` 换成 `-v easysearch-config:/c` 即可。
>
> 除 `jvm.options` 外，其它配置文件是否也要同步，按上面第 2 条命令的哈希比对结果判断；含自定义内容的
> 文件（如 `easysearch.yml`）不要直接覆盖。

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
  --health-cmd='curl -skf -u "admin:${EASYSEARCH_INITIAL_ADMIN_PASSWORD}" https://localhost:9200/_cluster/health || exit 1' \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=60s \
  -v easysearch-data:/app/easysearch/data \
  -v easysearch-config:/app/easysearch/config \
  -v easysearch-logs:/app/easysearch/logs \
  infinilabs/easysearch:{{< data "quick_start.easysearch.version" >}}
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
# 5. 用了 --user 指定运行用户 — 去掉 --user，容器需以默认 root 启动（entrypoint 用 gosu 降权）
# 6. 挂载的 config 目录为空 — 按"数据持久化到本地"第 3 步从镜像初始化 config
```

### 升级后容器无法启动

`docker logs` 中出现以下任一报错，通常是持久化的 `config` 目录里 `jvm.options` 与新版本镜像不匹配：

```
A command line option has attempted to allow or enable the Security Manager
java.lang.IllegalStateException: security agent is not loaded
```

按前文"数据持久化到本地"一节的第 5 步，用新版本镜像同步 `config/jvm.options` 后再重新运行容器。
**只修改单个 JVM 参数不够**，需整体同步该文件。

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
