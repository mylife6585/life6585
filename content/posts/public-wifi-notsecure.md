---
title: "Public Wifi SSL Notsecure"
date: 2021-11-05T17:37:16+08:00
draft: true
description: "公共wifi访问https报错"
tags: ["hsts","ssl"]
categories: ["devops"]
keywords: ["hsts","ssl"]
draft: false
isCJKLanguage: true
---

## 问题

https网站，星巴克、酒店等公共wifi情况下访问会报错：

你的连接不是私密连接,NET:ERR_CERT_AUTHORITY_INVALID

其它环境正常

## 原因

用https://www.ssllabs.com/ssltest/ 检测网站正常

网站启用了HSTS，本来是为http跳转到https这步更安全

但星巴克这种公共wifi 会有个登录页面，打断正常请求，导致浏览器就认为不安全


## 解决

关闭hsts

我们用的是nginx-ingress

添加nginx-configuration.yaml

```
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  name: nginx-configuration
  namespace: ingress-nginx
data:
  hsts: "false"
```

```
kubectl apply -f nginx-configuration.yaml
```

检测 https://www.ssleye.com/ssltool/hsts_check.html