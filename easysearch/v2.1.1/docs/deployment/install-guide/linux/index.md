---
title: "Linux"
date: 0001-01-01
summary: "Linux 环境下使用 Easysearch #   为了安全起见，Easysearch 不支持通过 root 身份来运行，需要新建普通用户，如 easysearch 用户来快速运行 Easysearch。
 一键安装 #   通过我们提供的自动安装脚本可自动下载最新版本的 easysearch 进行解压安装，默认解压到 /data/easysearch
 curl -sSL http://get.infini.cloud | bash -s -- -p easysearch  脚本的可选参数如下：
-v [版本号]（默认采用最新版本号）
-d [安装目录]（默认安装到/data/easysearch）
 bundle 包运行 #   bundle 是内置 JDK 的安装包，不需要额外下载 JDK，可直接解压运行。
 # 创建 easysearch 用户 groupadd -g 602 easysearch useradd -u 602 -g easysearch -m -d /home/easysearch -c &#39;easysearch&#39; -s /bin/bash easysearch # 创建 easysearch 安装目录 mkdir -p /data/easysearch # 下载 bundle 包并解压到安装目录 wget -O - https://release."
---


# Linux 环境下使用 Easysearch

> 为了安全起见，Easysearch 不支持通过 root 身份来运行，需要新建普通用户，如 `easysearch` 用户来快速运行 Easysearch。

## 一键安装

> 通过我们提供的自动安装脚本可自动下载最新版本的 easysearch 进行解压安装，默认解压到 /data/easysearch

```
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch
```

> 脚本的可选参数如下：  
> &nbsp;&nbsp;&nbsp;&nbsp;-v [版本号]（默认采用最新版本号）  
> &nbsp;&nbsp;&nbsp;&nbsp;-d [安装目录]（默认安装到/data/easysearch）

## bundle 包运行

> bundle 是内置 JDK 的安装包，不需要额外下载 JDK，可直接解压运行。

```bash
# 创建 easysearch 用户
groupadd -g 602 easysearch
useradd -u 602 -g easysearch -m -d /home/easysearch -c 'easysearch' -s /bin/bash easysearch
# 创建 easysearch 安装目录
mkdir -p /data/easysearch
# 下载 bundle 包并解压到安装目录
wget -O - https://release.infinilabs.com/easysearch/stable/bundle/easysearch-1.14.0-2228-linux-amd64-bundle.tar.gz | tar -zx -C /data/easysearch
# 初始化
cd /data/easysearch && bin/initialize.sh
# 调整目录权限
chown -R easysearch:easysearch /data/easysearch
# 运行 Easysearch
su easysearch -c "/data/easysearch/bin/easysearch -d -p pid"
# 停止 Easysearch
kill -9 $(cat pid)
```

## 手动安装

以 root 用户进行下面的操作

1. 下载 JDK

```bash
#下载JDK并存储到/usr/src目录
wget -N https://cdn.azul.com/zulu/bin/zulu17.54.21-ca-jre17.0.13-linux_x64.tar.gz -P /usr/src
```

2. 创建 JDK 解压后存储路径

```bash
mkdir -p /usr/local/jdk
```

3. 解压文件到创建好的目录

```bash
tar -zxf /usr/src/zulu*.tar.gz -C /usr/local/jdk --strip-components 1
```

4. 配置环境变量

```bash
#下载文件到/etc/profile.d
wget -N https://release.infinilabs.com/easysearch/archive/java.sh -P /etc/profile.d
```

5. 让配置生效

```bash
source /etc/profile
```

6. 检查 java 版本信息

```bash
java -version
```

7. 通过在线脚本进行 Easysearch 安装

```bash
curl -sSL http://get.infini.cloud |bash -s -- -p easysearch
```

8. 创建 easysearch 用户组

```bash
groupadd -g 602 easysearch
```

9. 创建 easysearch 用户，并添加到 easysearch 用户组

```bash
useradd -u 602 -g easysearch -m -d /home/easysearch -c "Easysearch user" -s /bin/bash easysearch
```

10. 将 JDK 放置或通过软链接到 /data/easysearch/jdk

```bash
ln -s /usr/local/jdk /data/easysearch/jdk
```

11. 初始化证书，密码及插件

```bash
cd /data/easysearch && bin/initialize.sh
```

12. 调整目录属主为 easysearch

```bash
chown -R easysearch:easysearch /data/easysearch
```

13. 切换到 easysearch 用户

```bash
su - easysearch
```

14. 运行 Easysearch

```bash
cd /data/easysearch && bin/easysearch
```

## 将 Easysearch 配置为服务

> 如果您想通过服务的方式来运行 Easysearch，可手工配置 Easysearch 服务文件

1. 下载服务文件

```bash
wget -N https://release.infinilabs.com/easysearch/archive/easysearch.service -P /usr/lib/systemd/system
```

或者直接编辑和执行下面的脚本:

```bash
sudo tee /usr/lib/systemd/system/easysearch.service <<-'EOF'
[Unit]
Description=Easysearch
Documentation=https://www.infinilabs.com
After=network.target

[Service]
Type=forking
User=easysearch
WorkingDirectory=/data/easysearch
Environment="ES_PATH_CONF=/data/easysearch/config"
PIDFile=/data/easysearch/pid
ExecStart=/data/easysearch/bin/easysearch -d -p pid
PrivateTmp=true
LimitNOFILE=65536
LimitNPROC=65536
LimitAS=infinity
LimitFSIZE=infinity
LimitMEMLOCK=infinity
TimeoutStopSec=0
KillSignal=SIGTERM
KillMode=process
SendSIGKILL=no
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
EOF
```

> 如果您的 Easysearch 运行用户及安装目录不同，请修改服务文件中的 User 及 ExecStart。

2. 重新加载服务配置文件

```bash
systemctl daemon-reload
```

3. 启动 Easysearch 服务

```bash
systemctl start easysearch
```

4. 检查 Easysearch 服务状态

```bash
systemctl status easysearch
```

后续验证工作，请继续查看[安装指南](./_index.md#验证工作)

