---
title: "前端访问阿里云cdn静态资源失败"
date: 2021-01-25T11:19:09+08:00
description: "前端访问阿里云cdn静态资源失败"
tags: ["alicloud","cdn"]
categories: ["devops"]
keywords: ["alicloud","cdn"]
draft: false
isCJKLanguage: true
---

## 背景：

前端服务器部署在阿里云ecs，静态资源部署在阿里云cdn，前端server通过proxy_pass访问阿里云CDN

## 现象：

https://app.creams.io/public/js/rewriteFunc/toFixed.js

静态资源无法访问



## 分析：

1. 怀疑CDN, OSS 存在问题

https://static.creams.io/web-master/public/js/rewriteFunc/toFixed.js

https://creams-static.oss-cn-hangzhou.aliyuncs.com/web-master/public/js/rewriteFunc/toFixed.js

测试正常

2. 怀疑前端服务器与阿里云CDN通信不正常，在前端服务器节点测试

https://static.creams.io/web-master/public/js/rewriteFunc/toFixed.js

https://creams-static.oss-cn-hangzhou.aliyuncs.com/web-master/public/js/rewriteFunc/toFixed.js

测试正常

3. 前端服务器本地请求

https://app.creams.io/public/js/rewriteFunc/toFixed.js

http://localhost:8889/public/js/rewriteFunc/toFixed.js

失败

查看nginx错误日志

>2021/01/25 01:38:15 [error] 7#7: *825 upstream timed out (110: Operation timed out) while connecting to upstream, client: 127.0.0.1, server: , request: "GET /public/js/rewriteFunc/toFixed.js HTTP/1.1", upstream: "https://114.80.187.99:443/web-master/public/js/rewriteFunc/toFixed.js", host: "localhost:8889"


114.80.187.99与本地PING解析不一致

发阿里云工单，确认114.80.187.99CDN节点已下线，但nginx dns cache导致一直请求已下线节点


## 解决

ngin 增加配置resovler，指向阿里云公共dns

>resolver 223.5.5.5 223.6.6.6;


## 参考

https://www.nadeau.tv/post/nginx-proxy_pass-dns-cache/



