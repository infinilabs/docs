---
title: "命令行工具参考"
date: 0001-01-01
summary: "命令行工具参考 #  Easysearch 提供了一组命令行工具，位于安装目录的 bin/ 目录下，用于启动服务、管理安全设置和安装插件等。
bin/easysearch — 启动服务 #  启动 Easysearch 节点。
语法 #  bin/easysearch [选项] 命令行选项 #     选项 说明     -d 以守护进程（后台）模式启动。不能与 --pidfile 单独使用时不写 PID 文件   -p &lt;路径&gt; 启动时将进程 ID (PID) 写入指定文件，便于通过 PID 管理进程   -q 关闭标准输出和标准错误的控制台日志输出。日志仍会写入日志文件   -E &lt;设置&gt;=&lt;值&gt; 覆盖配置文件中的设置。可多次使用以覆盖多个设置   -V 显示 Easysearch 版本信息并退出   -h, --help 显示帮助信息   -v, --verbose 显示详细输出   -s, --silent 只显示错误信息    使用示例 #  前台启动（适合开发调试）："
---


# 命令行工具参考

Easysearch 提供了一组命令行工具，位于安装目录的 `bin/` 目录下，用于启动服务、管理安全设置和安装插件等。

## bin/easysearch — 启动服务

启动 Easysearch 节点。

### 语法

```bash
bin/easysearch [选项]
```

### 命令行选项

| 选项 | 说明 |
|------|------|
| `-d` | 以守护进程（后台）模式启动。不能与 `--pidfile` 单独使用时不写 PID 文件 |
| `-p <路径>` | 启动时将进程 ID (PID) 写入指定文件，便于通过 PID 管理进程 |
| `-q` | 关闭标准输出和标准错误的控制台日志输出。日志仍会写入日志文件 |
| `-E <设置>=<值>` | 覆盖配置文件中的设置。可多次使用以覆盖多个设置 |
| `-V` | 显示 Easysearch 版本信息并退出 |
| `-h`, `--help` | 显示帮助信息 |
| `-v`, `--verbose` | 显示详细输出 |
| `-s`, `--silent` | 只显示错误信息 |

### 使用示例

**前台启动**（适合开发调试）：

```bash
bin/easysearch
```

**后台启动并记录 PID**：

```bash
bin/easysearch -d -p /var/run/easysearch.pid
```

**覆盖配置项启动**：

```bash
bin/easysearch -E cluster.name=my_cluster -E node.name=node_1
```

**查看版本**：

```bash
bin/easysearch -V
```

输出示例：

```
Version: 1.x.x, Build: xxxxxxx/..., JVM: xx.x.x
```

**使用自定义配置目录**：

```bash
ES_PATH_CONF=/opt/easysearch/config bin/easysearch
```

**通过 PID 文件停止后台进程**：

```bash
kill $(cat /var/run/easysearch.pid)
```

### -E 设置覆盖

`-E` 选项可以覆盖 `easysearch.yml` 中的任何设置，优先级高于配置文件：

```bash
# 覆盖集群名称和网络绑定地址
bin/easysearch \
  -E cluster.name=production \
  -E network.host=0.0.0.0 \
  -E discovery.seed_hosts=10.0.0.1,10.0.0.2 \
  -E node.name=node-1
```

> **注意**：通过 `-E` 传入的设置仅在当前进程生效，不会写入配置文件。

## bin/easysearch-keystore — 安全设置管理

管理 Easysearch 的安全密钥库（keystore），用于存储敏感配置（如密码、API 密钥等），详见 [Keystore 安全设置]({{< relref "keystore" >}})。

### 可用命令

| 命令 | 说明 |
|------|------|
| `create` | 创建新的密钥库 |
| `list` | 列出密钥库中的所有条目 |
| `add <名称>` | 添加字符串类型的设置 |
| `add-file <名称> <文件>` | 添加文件类型的设置 |
| `remove <名称>` | 从密钥库中移除指定设置 |
| `upgrade` | 升级密钥库格式 |
| `passwd` | 修改密钥库密码 |
| `has-passwd` | 检查密钥库是否设置了密码 |

## bin/easysearch-plugin — 插件管理

安装、卸载和管理 Easysearch 插件。

### 可用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `install <插件>` | 安装插件 | `bin/easysearch-plugin install analysis-ik` |
| `remove <插件>` | 卸载插件 | `bin/easysearch-plugin remove analysis-ik` |
| `list` | 列出已安装的插件 | `bin/easysearch-plugin list` |

### 安装示例

```bash
# 从内置仓库安装
bin/easysearch-plugin install analysis-ik

# 从本地文件安装
bin/easysearch-plugin install file:///path/to/plugin.zip

# 从 URL 安装
bin/easysearch-plugin install https://example.com/plugin.zip
```

> **注意**：安装或卸载插件后需要重启 Easysearch 节点才能生效。

## 设置优先级

当同一个设置在多处配置时，Easysearch 按以下优先级（从高到低）应用：

| 优先级 | 来源 | 说明 |
|--------|------|------|
| 1（最高） | Transient 集群设置 | 通过 `PUT _cluster/settings` 设置，重启后失效 |
| 2 | Persistent 集群设置 | 通过 `PUT _cluster/settings` 设置，持久保存 |
| 3 | 命令行 `-E` 选项 | 启动时通过 CLI 传入 |
| 4 | `easysearch.yml` 配置文件 | 静态配置文件 |
| 5（最低） | 默认值 | 代码内置的默认值 |

> **建议**：对于固定不变的配置（如路径、网络绑定），写在 `easysearch.yml` 中；对于需要动态调整的配置（如水位线、限流），通过集群设置 API 管理。

