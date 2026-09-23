---
title: "Windows"
date: 0001-01-01
summary: "Windows 环境下使用 Easysearch #  目前，有多种方案可以在 Windows 下体验 Easysearch。
前置要求 #   Windows 10 / Windows Server 2016 或更高版本 至少 4 GB 可用内存 JDK 11+（推荐 JDK 17+，2.0.3 及以上版本要求 JDK 21+）。Bundle 包已内置 JDK，无需单独安装。  方案一：Docker 安装（推荐） #  如果您的 Windows 环境上有 Docker Desktop，可以用最简单的方式启动：
docker run -d --name easysearch ` -p 9200:9200 ` -e &#34;EASYSEARCH_INITIAL_ADMIN_PASSWORD=EasysearchP@ssw0rd&#34; ` -e &#34;ES_JAVA_OPTS=-Xms512m -Xmx512m&#34; ` infinilabs/easysearch:latest  详细 Docker 配置请参考 Docker 环境下使用 Easysearch。
 方案二：手工安装（无 HTTPS） #   由于 Windows 环境下默认没有 OpenSSL，生成证书不太方便。如果仅用于开发测试，可以先关闭安全模块快速体验。生产环境请务必启用安全功能（参见方案三）。"
---


# Windows 环境下使用 Easysearch

目前，有多种方案可以在 Windows 下体验 Easysearch。

## 前置要求

- Windows 10 / Windows Server 2016 或更高版本
- 至少 4 GB 可用内存
- JDK 11+（推荐 JDK 17+，2.0.3 及以上版本要求 JDK 21+）。Bundle 包已内置 JDK，无需单独安装。

## 方案一：Docker 安装（推荐）

如果您的 Windows 环境上有 Docker Desktop，可以用最简单的方式启动：

```powershell
docker run -d --name easysearch `
  -p 9200:9200 `
  -e "EASYSEARCH_INITIAL_ADMIN_PASSWORD=EasysearchP@ssw0rd" `
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" `
  infinilabs/easysearch:latest
```

> 详细 Docker 配置请参考 [Docker 环境下使用 Easysearch]({{< relref "./docker.md" >}})。

## 方案二：手工安装（无 HTTPS）

> 由于 Windows 环境下默认没有 OpenSSL，生成证书不太方便。如果仅用于开发测试，可以先关闭安全模块快速体验。生产环境**请务必启用安全功能**（参见方案三）。

1. 手工下载 [Easysearch](https://release.infinilabs.com/easysearch/stable/easysearch-{{< data "quick_start.easysearch.version" >}}-windows-amd64.zip)，解压到目标目录（如 `D:\easysearch`）。
2. 手工下载 [JDK](https://release.infinilabs.com/easysearch/jdk/21/graalvm-jdk-21_windows-x64_bin.zip)，解压到 Easysearch 安装目录下，并将目录名称重命名为 `jdk`。

> 也可以下载 Bundle 包（内置 JDK），省去手动配置 JDK 的步骤。Bundle 包[下载地址](https://release.infinilabs.com/easysearch/stable/bundle/)。

3. 用记事本打开 `config\easysearch.yml`，修改配置：

```yaml
# 关闭安全模块（仅开发测试，生产环境不建议）
security.enabled: false
```

4. 双击运行 `bin\easysearch.bat` 或在命令行执行：

```bat
bin\easysearch.bat
```

5. 验证安装：

```powershell
# 在 PowerShell 中
Invoke-RestMethod -Uri "http://localhost:9200"

# 或在浏览器中打开 http://localhost:9200
```

## 方案三：通过 Git Bash 安装（支持 HTTPS）

安装 [Git for Windows](https://git-scm.com/download/win)，使用其内置的 Bash 环境来执行初始化脚本，可正常生成证书并启用 HTTPS。

由于 windows 环境下，Git Bash 自带的 curl 与 liunx 不兼容，可能会导致初始化脚本生成证书格式不匹配等兼容性问题，建议下载  [curl for Windows](https://curl.se/windows/) ，并替换 Git Bash 自带的 curl。

> 注意：以下操作在 **Git Bash** 终端中执行。

1. 下载 [curl for Windows](https://curl.se/windows/) 

```bash
# 下载并解压 [curl for Windows]，假设解压后路径：/xxx/curl-xx/bin/curl.exe （具体路径请以本地环境为准）
# 执行 curl.exe 文件覆盖替换
sudo cp /xxx/curl-xx/bin/curl.exe /mingw64/bin/curl.exe
```

2. 通过在线脚本安装 Easysearch

```bash
curl -sSL http://get.infini.cloud | bash -s -- -p easysearch -d /d/data/easysearch
```

3. 下载并配置 JDK

```bash
# 下载 JDK
curl -# https://release.infinilabs.com/easysearch/jdk/21/graalvm-jdk-21_windows-x64_bin.zip -o /d/data/easysearch/jdk.zip

# 解压并重命名
cd /d/data/easysearch && unzip -q jdk.zip
mv graalvm* jdk

# 设置环境变量
export JAVA_HOME=/d/data/easysearch/jdk
```

4. 初始化证书、密码及插件

```bash
bin/initialize.sh
```

> 初始化过程中会生成随机密码，只会在终端显示一次，请妥善保存。

5. 运行 Easysearch

```bat
bin\easysearch.bat
```

6. 验证安装

```bash
# 在 Git Bash 中（使用初始化时输出的密码）
curl -ku admin:YOUR_PASSWORD https://localhost:9200
```

## 常见问题

### 初始化脚本报错

执行初始化脚本 `bin/initialize.sh` 报错如下：

```
错误：找不到或无法加载主类 com.infinilabs.securlty.tools.Hasner
原因: java.lang.ClassNotFoundException: com.infinilabs.security.tools.Hasher
```
可能原因是本机系统自带的 curl.exe 与脚本生成证书的格式兼容问题。解决办法是下载最新版 [curl for Windows](https://curl.se/windows/) 并替换 Git Bash 自带的 curl.exe （自带 curl 程序路径通常位于：/mingw64/bin/curl.exe）

### 端口被占用

如果 9200 端口被占用，可修改 `config\easysearch.yml`：

```yaml
http.port: 9201
```

### 忘记 admin 密码

在 Git Bash 中运行：

```bash
cd /d/data/easysearch
bin/reset_admin_password.sh
```

### 内存不足

编辑 `config\jvm.options`，调小堆内存：

```
-Xms512m
-Xmx512m
```

## 延伸阅读

- [Docker 部署]({{< relref "./docker.md" >}})
- [测试环境部署]({{< relref "./test-env.md" >}})
- [系统调优]({{< relref "/docs/deployment/config/settings.md" >}})

