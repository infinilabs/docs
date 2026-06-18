---
title: "网络配置"
date: 0001-01-01
summary: "网络配置 #  本页介绍 easysearch.yml 中与网络绑定、端口和 HTTP 行为相关的配置项。这些都是静态设置，修改后需要重启节点生效。
 核心网络参数 #  network.host #  network.host: 0.0.0.0    项目 说明     参数 network.host   默认值 _local_（仅本机 127.0.0.1）   属性 静态   说明 节点绑定的主机名或 IP 地址。同时设置 network.bind_host 和 network.publish_host。这是最常修改的网络配置    特殊值：
   值 含义     _local_ 回环地址（127.0.0.1），仅本机可访问   _site_ 内网地址（如 192.168.x.x / 10.x.x.x）   _global_ 公网地址   0."
---


# 网络配置

本页介绍 `easysearch.yml` 中与网络绑定、端口和 HTTP 行为相关的配置项。这些都是**静态设置**，修改后需要重启节点生效。

---

## 核心网络参数

### network.host

```yaml
network.host: 0.0.0.0
```

| 项目 | 说明 |
|------|------|
| **参数** | `network.host` |
| **默认值** | `_local_`（仅本机 127.0.0.1） |
| **属性** | 静态 |
| **说明** | 节点绑定的主机名或 IP 地址。同时设置 `network.bind_host` 和 `network.publish_host`。这是最常修改的网络配置 |

**特殊值**：

| 值 | 含义 |
|------|------|
| `_local_` | 回环地址（127.0.0.1），仅本机可访问 |
| `_site_` | 内网地址（如 192.168.x.x / 10.x.x.x） |
| `_global_` | 公网地址 |
| `0.0.0.0` | 绑定所有网卡接口 |
| 具体 IP | 绑定到指定 IP 地址 |

> **重要**：一旦将 `network.host` 设为非 `_local_` 的值，Easysearch 将进入"生产模式"，执行更严格的启动检查（bootstrap checks）。

### network.bind_host

```yaml
network.bind_host: 0.0.0.0
```

| 项目 | 说明 |
|------|------|
| **参数** | `network.bind_host` |
| **默认值** | 随 `network.host` |
| **属性** | 静态 |
| **说明** | 节点监听传入请求的具体地址。一般情况下只需设置 `network.host`，仅在需要区分绑定和通告地址时使用 |

### network.publish_host

```yaml
network.publish_host: 192.168.1.10
```

| 项目 | 说明 |
|------|------|
| **参数** | `network.publish_host` |
| **默认值** | 随 `network.host` |
| **属性** | 静态 |
| **说明** | 节点向集群中其他节点通告的地址。在多网卡环境中，应显式设置为其他节点可达的内网 IP |

**何时需要单独设置**：

- 服务器有多个网卡（如内网 + 外网）
- 运行在 Docker 中需要通告宿主机 IP
- NAT 环境下需要通告映射后的地址

**示例（多网卡）**：

```yaml
network.bind_host: 0.0.0.0          # 监听所有接口
network.publish_host: 192.168.1.10  # 通告内网地址
```

---

## HTTP 端口

### http.port

```yaml
http.port: 9200
```

| 项目 | 说明 |
|------|------|
| **参数** | `http.port` |
| **默认值** | `9200-9300` |
| **属性** | 静态 |
| **说明** | HTTP REST API 监听端口。客户端通过此端口与 Easysearch 交互。支持单个值或端口范围 |

**端口范围行为**：指定范围时（如 `9200-9300`），节点将绑定到该范围内的第一个可用端口。生产环境建议设置为固定端口。

### transport.port

```yaml
transport.port: 9300
```

| 项目 | 说明 |
|------|------|
| **参数** | `transport.port` |
| **默认值** | `9300-9400` |
| **属性** | 静态 |
| **说明** | 节点间内部通信（Transport 层）端口。用于集群内节点间的数据传输、分片恢复、集群状态同步等。客户端不会直接连接此端口 |

> **注意**：在所有具有 master 资格的节点上，请将 `transport.port` 设置为单个固定值，避免因端口漂移导致发现失败。

---

## HTTP 调优参数

### 请求大小限制

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `http.max_content_length` | `100mb` | HTTP 请求体最大大小。超过此大小的请求将被拒绝。如果有大量 bulk 写入需求，可适当调大 |
| `http.max_header_size` | `8kb` | HTTP 请求头（header）最大大小 |
| `http.max_initial_line_length` | `4kb` | HTTP 初始行（URL）最大长度。如果使用超长查询字符串，可能需要调大 |

**示例（大 bulk 写入场景）**：

```yaml
http.max_content_length: 200mb
```

### 压缩

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `http.compression` | `true` | 是否启用 HTTP 响应压缩（gzip）。启用可以减少网络传输量，但会增加少量 CPU 开销 |

### CORS（跨域资源共享）

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `http.cors.enabled` | `false` | 是否启用 CORS。仅在浏览器前端直接访问 Easysearch API 时需要开启 |
| `http.cors.allow-origin` | — | 允许的 CORS 来源。支持正则表达式。`"*"` 表示允许所有来源（不推荐用于生产环境） |
| `http.cors.allow-methods` | `OPTIONS, HEAD, GET, POST, PUT, DELETE` | 允许的 HTTP 方法 |
| `http.cors.allow-headers` | `X-Requested-With, Content-Type, Content-Length` | 允许的自定义请求头 |
| `http.cors.allow-credentials` | `false` | 是否允许发送凭据（cookies） |
| `http.cors.max-age` | `1728000`（20 天） | 预检请求（preflight）的缓存时间（秒） |

**示例（允许特定域名）**：

```yaml
http.cors.enabled: true
http.cors.allow-origin: "/https?://localhost(:[0-9]+)?/"
```

**示例（允许所有来源，仅用于开发）**：

```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: "X-Requested-With, Content-Type, Content-Length, Authorization"
```

---

## 网络参数速查表

| 参数 | 默认值 | 属性 | 说明 | 生产环境建议 |
|------|--------|------|------|-------------|
| `network.host` | `_local_` | 静态 | 绑定地址 | 设为具体内网 IP |
| `network.bind_host` | 随 `network.host` | 静态 | 监听地址 | 多网卡时配合使用 |
| `network.publish_host` | 随 `network.host` | 静态 | 通告地址 | 多网卡/Docker 需显式设置 |
| `http.port` | `9200-9300` | 静态 | HTTP 端口 | 设为固定值 `9200` |
| `transport.port` | `9300-9400` | 静态 | 节点通信端口 | 设为固定值 `9300` |
| `http.max_content_length` | `100mb` | 静态 | 请求体上限 | 大 bulk 场景可调至 `200mb` |
| `http.compression` | `true` | 静态 | HTTP 压缩 | 保持默认 |
| `http.cors.enabled` | `false` | 静态 | CORS 开关 | 按需开启 |

---

## 配置示例

### 生产集群（单网卡）

```yaml
network.host: 192.168.1.10
http.port: 9200
transport.port: 9300
```

### 生产集群（多网卡 / Docker）

```yaml
network.bind_host: 0.0.0.0
network.publish_host: 192.168.1.10
http.port: 9200
transport.port: 9300
```

### 开发环境（本机访问）

```yaml
network.host: 0.0.0.0
http.port: 9200
transport.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
```

---

## 延伸阅读

- [集群发现]({{< relref "./discovery.md" >}}) — 节点间如何发现彼此
- [安全配置]({{< relref "./security-settings.md" >}}) — HTTPS（TLS）配置
- [系统调优]({{< relref "../settings.md" >}}) — TCP 网络内核参数

