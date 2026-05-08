---
title: "信创平台安装"
date: 0001-01-01
summary: "信创平台安装 #  目前，Easysearch 已支持在部分国产操作系统上运行，如龙芯、兆芯、申威、鲲鹏、海光、飞腾、麒麟软件、统信 UOS 、移动云 BC-Linux等，联网环境建议使用一键安装脚本进行安装，离线环境建议下载 Bundle 包进行安装，如果您有其他国产操作系统的安装需求，欢迎通过 提交工单与我们联系。
 前提条件：已参照 系统调优进行了系统优化，同时为 Easysearch 创建了专用的用户。
 初始化系统参数及用户命令参考：
# 调整内核配置 echo &#34;vm.max_map_count=262144&#34; &gt;&gt; /etc/sysctl.conf &amp;&amp; sysctl -p # 增加用户组与用户 groupadd -r easysearch &amp;&amp; useradd -r -g easysearch -d /home/easysearch -s /sbin/nologin -c &#34;Easysearch Service Account&#34; easysearch 以下是各个平台的安装参考：
  龙芯平台安装参考  兆芯平台安装参考  申威平台安装参考  鲲鹏平台安装参考  海光平台安装参考  飞腾平台安装参考  麒麟软件平台安装参考  统信 UOS 平台安装参考  移动云 BC-Linux 平台安装参考  "
---


# 信创平台安装

目前，Easysearch 已支持在部分国产操作系统上运行，如龙芯、兆芯、申威、鲲鹏、海光、飞腾、麒麟软件、统信 UOS 、移动云 BC-Linux等，联网环境建议使用一键安装脚本进行安装，离线环境建议下载 [Bundle 包](https://release.infinilabs.com/easysearch/stable/bundle/)进行安装，如果您有其他国产操作系统的安装需求，欢迎通过[提交工单](https://www.infinilabs.cn/company/contact/)与我们联系。

> 前提条件：已参照[系统调优]({{< relref "/docs/deployment/config/settings" >}})进行了系统优化，同时为 Easysearch 创建了专用的用户。

初始化系统参数及用户命令参考：

```bash
# 调整内核配置
echo "vm.max_map_count=262144" >> /etc/sysctl.conf && sysctl -p
# 增加用户组与用户
groupadd -r easysearch && useradd -r -g easysearch -d /home/easysearch -s /sbin/nologin -c "Easysearch Service Account" easysearch
```


以下是各个平台的安装参考：
- [龙芯平台安装参考](./loongson.md)
- [兆芯平台安装参考](./zhaoxin.md)
- [申威平台安装参考](./shenwei.md)
- [鲲鹏平台安装参考](./kunpeng.md)
- [海光平台安装参考](./hygon.md)
- [飞腾平台安装参考](./feiteng.md)
- [麒麟软件平台安装参考](./kylin.md)
- [统信 UOS 平台安装参考](./uniontech.md)
- [移动云 BC-Linux 平台安装参考](./bclinux.md)