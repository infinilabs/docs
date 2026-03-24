---
title: "系统调优"
date: 0001-01-01
summary: "系统调优 #  芯片及操作系统兼容性 #  目前已在国产主流芯片及操作系统上进行了验证，分别为 openEuler、统信 UOS、麒麟、龙芯、申威、兆芯。同样也兼容 Windows、 MacOS、 CentOS、 Ubuntu、 RedHat 等常用操作系统。
Java 兼容性 #  默认情况下 Easysearch 并不包含 JDK, 推荐使用 Java 15.0.1+9 或 Java 17.0.6+10, 最低版本要求为 Java 11, 要使用不同的 Java 安装，请将 JAVA_HOME 环境变量设置为 Java 安装位置或将 JDK 软链接到 Easysearch 安装目录下取名为 jdk。
例如：
#设置 JAVA_HOME 环境变量，可放入 ~/.bashrc 或 /etc/profile export JAVA_HOME=/usr/local/jdk #软链接 sudo ln -s /usr/local/jdk /data/easysearch/jdk 网络要求 #  Easysearch 需要打开以下端口:
   端口 模块说明     9200 REST API   9300 节点间通信    系统参数 #  要保证 Easysearch 运行在最佳状态，其所在服务器的操作系统也需要进行相应的调优，以 Linux 为例。"
---


# 系统调优

## 芯片及操作系统兼容性

目前已在国产主流芯片及操作系统上进行了验证，分别为 openEuler、统信 UOS、麒麟、龙芯、申威、兆芯。同样也兼容 Windows、 MacOS、 CentOS、 Ubuntu、 RedHat 等常用操作系统。

## Java 兼容性

默认情况下 Easysearch 并不包含 JDK, 推荐使用 Java 15.0.1+9 或 Java 17.0.6+10, 最低版本要求为 Java 11, 要使用不同的 Java 安装，请将 JAVA_HOME 环境变量设置为 Java 安装位置或将 JDK 软链接到 Easysearch 安装目录下取名为 `jdk`。

例如：

```bash
#设置 JAVA_HOME 环境变量，可放入 ~/.bashrc 或 /etc/profile
export JAVA_HOME=/usr/local/jdk
#软链接
sudo ln -s /usr/local/jdk /data/easysearch/jdk
```

## 网络要求

Easysearch 需要打开以下端口:

| 端口 | 模块说明   |
| ---- | ---------- |
| 9200 | REST API   |
| 9300 | 节点间通信 |

## 系统参数

要保证 Easysearch 运行在最佳状态，其所在服务器的操作系统也需要进行相应的调优，以 Linux 为例。


```bash
sudo tee /etc/security/limits.d/21-infini.conf <<-'EOF'
*                soft    nofile         1048576
*                hard    nofile         1048576
*                soft    memlock        unlimited
*                hard    memlock        unlimited
root             soft    nofile         1048576
root             hard    nofile         1048576
root             soft    memlock        unlimited
root             hard    memlock        unlimited
EOF
```

## 内核调优


```bash
cat << SETTINGS | sudo tee /etc/sysctl.d/70-infini.conf
fs.file-max = 10485760
fs.nr_open = 10485760
vm.max_map_count = 262145

net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535
net.core.rmem_default = 262144
net.core.wmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_max = 4194304

net.ipv4.ip_forward = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_tw_buckets = 300000
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_synack_retries = 0
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_time = 900
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_max_orphans = 131072
net.ipv4.tcp_rmem = 4096 4096 16777216
net.ipv4.tcp_wmem = 4096 4096 16777216
net.ipv4.tcp_mem = 786432 3145728 4194304
SETTINGS
```


执行下面的命令验证配置参数是否合法。

```bash
sysctl -p /etc/sysctl.d/70-infini.conf
```

> 您也可以重启操作系统，然后检查配置是否生效。

