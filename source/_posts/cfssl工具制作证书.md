---
title: cfssl工具制作证书
date: 2025-07-01 13:35:02
tags:
- ssl
categories:
- ssl
---

# **cfssl工具制作证书**

## 安装cfssl

[去git下载二进制文件]: https://github.com/cloudflare/cfssl/releases	"点击下载文件"

下载文件 

```
cfssl_1.6.5_linux_amd64   cfssl-certinfo_1.6.5_linux_amd64    cfssljson_1.6.5_linux_amd64
```



```java
# 赋权限
$ chmod +x cffs*
# 移动到 /usr/local/bin
$ mv cfssl_1.6.1_linux_amd64 /usr/local/bin/cfssl
$ mv cfssl-certinfo_1.6.1_linux_amd64 /usr/local/bin/cfssl-certinfo
$ mv cfssljson_1.6.1_linux_amd64 /usr/local/bin/cfssljson
```

## 生成根证书

```
# 根证书请求配置文件
$ vim ca-csr.json

{
        "CN": "ltdw",
        "key": {
                "algo": "rsa",
                "size": 2048
        },
        "names": [{
                "C": "CN",
                "ST": "Beijing",
                "L": "beijing",
                "O": "ltdw",
                "OU": "ltdw"
        }]
}

# 生成证书
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca

# -base ca 指定生成的证书名称
```

`CN` 证书名称

`C` Country， 国家

`L` Locality，地区，城市

`O` Organization Name，组织名称，公司名称

`OU` Organization Unit Name，组织单位名称，公司部门

`ST` State，州，省

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-08-01_16-25-38.jpg)

## 通过ca签发服务器证书

```
# 服务器证书请求配置文件
vim 192.168.1.101-csr.json

{
    "CN": "192.168.1.101",
    "hosts": [
      "192.168.1.101",
      "kube-master"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "ST": "BeiJing",
            "O": "ltdw",
            "OU": "ltdw"
        }
    ]
}


# 生成证书策略文件
$ cat ca-config.json
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "server": {
        "expiry": "87600h",
        "usages": [
          "signing",
          "key encipherment",
          "server auth",
          "client auth"
        ]
      },
      "intermediate": {
        "expiry": "87600h",
        "usages": [
          "signing",
          "key encipherment",
          "cert sign",
          "crl sign",
          "server auth",
          "client auth"
        ]
      }
    }
  }
}


# 生成证书
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=intermediate 192.168.1.101-csr.json | cfssljson -bare 192.168.1.101
# 单独生成csr文件
$ cfssl genkey csr.json | cfssljson -bare 192.168.1.101
# 证书和私钥合并成pfx
$ openssl pkcs12 -export -out certificate.pfx -inkey private_key.pem -in certificate.pem -certfile intermediate_cert.pem
# pfx 提取证书和私钥
$ openssl pkcs12 -in certificate.pfx -nocerts -nodes -out private_key.pem
$ openssl pkcs12 -in certificate.pfx -clcerts -nokeys -out certificate.pem
$ openssl verify -CAfile ca.pem intermediate.pem 
```

- 默认策略，指定了证书的有效期是一年(8760h)
- `usages.signing`, 表示该证书可用于签名其它证书；生成的 ca.pem 证书中 CA=TRUE
- `usages.server auth`：表示 client 可以用该 CA 对 server 提供的证书进行验证

- `usages.client auth`：表示 server 可以用该 CA 对 client 提供的证书进行验证

## 证书格式转换

```
# pem > crt
$ openssl x509 -in ca.pem -out ca.crt

# pem > key
$ openssl rsa -in ca.pem -out ca.key
```



## 挂载

在Linux操作系统中, 把192.168.1.101-key.pem和192.168.1.101.pem 挂载到nginx下

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-07-01_11-26-30.jpg)

本地计算机安装ca.crt证书 双击-安装证书-本地计算机-将所有证书都放入下列存储-受信任的根证书颁发机构

![](https://raw.githubusercontent.com/zhangzc-hub/img/main/img/Snipaste_2025-07-01_14-11-48.jpg)
