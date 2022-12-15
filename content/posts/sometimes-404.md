---
title: "Sometimes 404"
date: 2022-12-15T14:22:55+08:00
draft: false
description: "小概率访问静态资源404"
tags: ["nginx","qos"]
categories: ["devops"]
keywords: ["nginx","qos"]
isCJKLanguage: true
---


## 现象

nginx反向代理请求到阿里云oss,

静态资源请求小概率404

直接请求oss 静态资源 没问题


##  分析

0. 错误日志


>2022/12/15 02:52:10 [error] 44#44: *9122 connect() failed (111: Connection refused) while connecting to upstream, client: 172.18.162.129, server: , request: "GET /dist/bundle.1670.4b095fc535ccd1d37d6f.js HTTP/1.1", upstream: "https://113.96.179.225:443/web-xzlzs/dist/bundle.1670.4b095fc535ccd1d37d6f.js", host: "yxzl-test.yuexiuproperty.cn", referrer: "https://yxzl-test.yuexiuproperty.cn/?code=lhGGZ2vmuyTzOwpD4t6ToM4t1tOTyfn6&state=1671010519736"
>2022/12/15 02:52:10 [error] 44#44: *9122 open() "/usr/share/nginx/html/50x.html" failed (2: No such file or directory), client: 172.18.162.129, server: , request: "GET /dist/bundle.1670.4b095fc535ccd1d37d6f.js HTTP/1.1", upstream: "https://113.96.179.225:443/web-xzlzs/dist/bundle.1670.4b095fc535ccd1d37d6f.js", host: "yxzl-test.yuexiuproperty.cn", referrer: "https://yxzl-test.yuexiuproperty.cn/?code=lhGGZ2vmuyTzOwpD4t6ToM4t1tOTyfn6&state=1671010519736"


404 不是直接原因

1. 反向代理请求 出现错误

不同的域名解析导致？ 多个cdn地址？

测试不是该原因

2. 资源限制？

服务端 请求限制？

同样的镜像，本地跑起来测试正常，不是阿里云的限制


客户端

怀疑qos限制，去除cpu、内存限制正常

中间

未测试

## 解决

```
        resources:
          limits:
            cpu: "1"
            memory: 1000Mi
          requests:
            cpu: 100m
            memory: 50Mi
```

改成

```
        resources:
          limits:
            cpu: "1"
            memory: 1000Mi
          requests:
            cpu: 100m
            memory: 150Mi
```
