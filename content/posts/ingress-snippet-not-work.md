---
title: "Ingress Snippet Not Work"
date: 2021-07-16T17:10:26+08:00
description: "Ingress snippet配置不生效"
tags: ["k8s","ingress"]
categories: ["virtualization"]
keywords: ["k8s","ingress"]
draft: false
isCJKLanguage: true
---

# 现象

ingress snippet 配置之后不生效，如下
增加了一个验证/jOIZN4meSj.txt 的配置，但访问不生效
老的/actuator确实返回403

```
    nginx.ingress.kubernetes.io/server-snippet: |
        location = /jOIZN4meSj.txt {
           return 200 "17bc5900ff3e50d67ba89514317ac1d0";
        }
        location ~ /actuator {
            deny all;
        }
```


# 定位

1.  查看nginx-ingress pod 中 的nginx.conf 配置

对应的域名下面确实没有新增 

```
        location = /jOIZN4meSj.txt {
           return 200 "17bc5900ff3e50d67ba89514317ac1d0";
        }
```

2.  查看 nginx-ingress pod  的日志

```
W0716 08:31:18.554066       6 controller.go:1089] Server snippet already configured for server "rc-accounts.test.io", skipping (Ingress "test/rc-auth-ingress")
```

确定是因为多个yaml 中，包含重复的snippet 配置，导致后面的配置被skip


# 修复

同一个域名的snippet 整理到一个yaml 文件中
