---
title: "证书管理"
date: 0001-01-01
summary: "证书管理 #  使用了 cert-manager 进行自动化管理证书，对于过期证书会自动重新颁发。
在这里我们根据 cert-manager 官方的配置方式配置了3套 Certificate 证书：ca-certificate、easysearch-certs 和 easysearch-admin-certs，分别用于节点间证书、http 访问证书和admin 管理员证书，具体参考下属 yaml 文件，重点需要主要证书的有效期(duration 字段)、更新时间(renewBefore 字段)和 commonName(infinilabs) 字段。
展开查看完整代码 apiVersion: cert-manager.io/v1 kind: Issuer metadata: name: selfsigned-issuer namespace: default spec: selfSigned: {} --- apiVersion: cert-manager.io/v1 kind: Certificate metadata: name: ca-certificate namespace: default spec: secretName: ca-cert duration: 9000h # ~1year renewBefore: 360h # 15d commonName: infinilabs isCA: true privateKey: size: 2048 usages: - digital signature - key encipherment issuerRef: name: selfsigned-issuer --- apiVersion: cert-manager."
---


# 证书管理

使用了 [cert-manager](https://github.com/cert-manager/cert-manager) 进行自动化管理证书，对于过期证书会自动重新颁发。  

在这里我们根据 cert-manager 官方的配置方式配置了3套 Certificate 证书：ca-certificate、easysearch-certs 和 easysearch-admin-certs，分别用于节点间证书、http 访问证书和admin 管理员证书，具体参考下属 yaml 文件，重点需要主要证书的有效期(duration 字段)、更新时间(renewBefore 字段)和 commonName(infinilabs) 字段。

{{< details "展开查看完整代码" "..." >}}
```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
  namespace: default
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: ca-certificate
  namespace: default
spec:
  secretName: ca-cert
  duration: 9000h # ~1year
  renewBefore: 360h # 15d
  commonName: infinilabs
  isCA: true
  privateKey:
    size: 2048
  usages:
    - digital signature
    - key encipherment
  issuerRef:
    name: selfsigned-issuer
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: ca-issuer
  namespace: default
spec:
  ca:
    secretName: ca-cert
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: easysearch-certs
  namespace: default
spec:
  secretName: easysearch-certs
  duration: 9000h # ~1year
  renewBefore: 360h # 15d
  isCA: false
  privateKey:
    size: 2048
    algorithm: RSA
    encoding: PKCS8
  dnsNames:
    - threenodes
    - threenodes-masters-0
    - threenodes-masters-1
    - threenodes-masters-2
    - threenodes-masters-3
    - threenodes-masters-4
    - threenodes-bootstrap-0
  usages:
    - signing
    - key encipherment
    - server auth
    - client auth
  commonName: infinilabs
  issuerRef:
    name: ca-issuer
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: easysearch-admin-certs
  namespace: default
spec:
  secretName: easysearch-admin-certs
  duration: 9000h # ~1year
  renewBefore: 360h # 15d
  isCA: false
  privateKey:
    size: 2048
    algorithm: RSA
    encoding: PKCS8
  commonName: infinilabs
  usages:
    - signing
    - key encipherment
    - server auth
    - client auth
  issuerRef:
    name: ca-issuer
```
{{< /details >}}

可以在证书所在目录查看证书的有效期
```shell
openssl x509 -in tls.crt -dates -noout

notBefore=Feb 23 16:02:03 2024 GMT
notAfter=Mar  4 16:02:03 2025 GMT
```

