---
title: "IPv6 支持"
date: 0001-01-01
summary: "Easysearch 的 IPv6 配置 #  配置 #  Easysearch 支持运行在 IPv6 模式，以下是具体操作：
假设本机 IPv6 地址为：fe80::18df:9883:1e27:b040%en0
修改 config/easysearch.yml 将 ip 相关的参数修改为 IPv6 对应的格式:
network.host: [&#34;::1&#34;, &#34;fe80::18df:9883:1e27:b040%en0&#34;] http.port: 9200 transport.port: 9300 discovery.seed_hosts: [&#34;[::1]:9300&#34;, &#34;[fe80::18df:9883:1e27:b040%en0]:9300&#34;] cluster.initial_master_nodes: [&#34;[fe80::18df:9883:1e27:b040%en0]:9300&#34;] 这个配置表示节点使用 IPv6 地址加入集群的配置，监听本地回环地址和一个特定的链路本地地址。
验证 #  启动 Easysearch，即可使用 curl 工具进行测试：
% curl -6 -v -ku &#34;admin:=juNrz?BY4SeSlL%Tm29OfP5&#34; &#34;https://[fe80::18df:9883:1e27:b040%en0]:9200&#34; * Trying fe80::18df:9883:1e27:b040:9200... * Connected to fe80::18df:9883:1e27:b040 (fe80::18df:9883:1e27:b040) port 9200 (#0) * ALPN, offering h2 * ALPN, offering http/1."
---


# Easysearch 的 IPv6 配置

## 配置

Easysearch 支持运行在 IPv6 模式，以下是具体操作：

假设本机 IPv6 地址为：`fe80::18df:9883:1e27:b040%en0`

修改 `config/easysearch.yml` 将 ip 相关的参数修改为 IPv6 对应的格式:

```auto
network.host: ["::1", "fe80::18df:9883:1e27:b040%en0"]
http.port: 9200
transport.port: 9300
discovery.seed_hosts: ["[::1]:9300", "[fe80::18df:9883:1e27:b040%en0]:9300"]
cluster.initial_master_nodes: ["[fe80::18df:9883:1e27:b040%en0]:9300"]

```
这个配置表示节点使用 IPv6 地址加入集群的配置，监听本地回环地址和一个特定的链路本地地址。

## 验证

启动 Easysearch，即可使用 curl 工具进行测试：

```auto
% curl -6 -v -ku "admin:=juNrz?BY4SeSlL%Tm29OfP5" "https://[fe80::18df:9883:1e27:b040%en0]:9200"
*   Trying fe80::18df:9883:1e27:b040:9200...
* Connected to fe80::18df:9883:1e27:b040 (fe80::18df:9883:1e27:b040) port 9200 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*  CAfile: /etc/ssl/cert.pem
*  CApath: none
* (304) (OUT), TLS handshake, Client hello (1):
* (304) (IN), TLS handshake, Server hello (2):
* (304) (IN), TLS handshake, Unknown (8):
* (304) (IN), TLS handshake, Request CERT (13):
* (304) (IN), TLS handshake, Certificate (11):
* (304) (IN), TLS handshake, CERT verify (15):
* (304) (IN), TLS handshake, Finished (20):
* (304) (OUT), TLS handshake, Certificate (11):
* (304) (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / AEAD-AES256-GCM-SHA384
* ALPN, server did not agree to a protocol
* Server certificate:
*  subject: C=IN; ST=FI; L=NI; O=ORG; OU=UNIT; CN=infini.cloud
*  start date: Sep 14 01:55:46 2025 GMT
*  expire date: Sep 12 01:55:46 2035 GMT
*  issuer: C=IN; ST=FI; L=NI; O=ORG; OU=UNIT; CN=infini.cloud
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Server auth using Basic with user 'admin'
> GET / HTTP/1.1
> Host: [fe80::18df:9883:1e27:b040]:9200
> Authorization: Basic YWRtaW46PWp1TnJ6P0JZNFNlU2xMJVRtMjlPZlA1
> User-Agent: curl/7.79.1
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< content-type: application/json; charset=UTF-8
< content-length: 563
<
{
  "name" : "zhangleideMacBook-Pro.local",
  "cluster_name" : "easysearch",
  "cluster_uuid" : "8KCQu9tjQcin1grPYhIbXw",
  "version" : {
    "distribution" : "easysearch",
    "number" : "1.15.2",
    "distributor" : "INFINI Labs",
    "build_hash" : "41dc40773ee4e12fe81d414f0e07cb3d3a64b7a5",
    "build_date" : "2025-09-18T01:13:46.174243Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.4",
    "minimum_wire_lucene_version" : "7.7.0",
    "minimum_lucene_index_compatibility_version" : "7.7.0"
  },
  "tagline" : "You Know, For Easy Search!"
}
* Connection #0 to host fe80::18df:9883:1e27:b040 left intact

```
-6 参数表示使用 IPv6，输出表示连接完全成功。

