---
title: "Yapi Ssl"
date: 2020-11-17T10:56:12+08:00
description: "解决Yapi模拟测试SSL不兼容问题"
tags: ["yapi","ssl"]
categories: ["devops"]
keywords: ["yapi","ssl"]
draft: false
isCJKLanguage: true
---

## 问题：

yapi 模仿请求https://www.alipay.com 的时候报错：

write EPROTO 140447099377472:error:1408F10B:SSL routines:ssl3_get_record:wrong version number:ssl/record/ssl3_record.c:332:

直接请求没有问题

另外请求https://www.baidu.com的时候没有问题

## 解决：

使用https://www.ssllabs.com/ssltest/对比了一下两个站点，发现最低ssl版本要求不一样

怀疑浏览器兼容tls1.2,但yapi用nodejs 作为客户端请求https://www.alipay.com 的过程中协商 tls协议出问题

先怀疑nodejs 10 不支持tls 1.3 导致，升级至 nodej 12，未解决问题

设置NODE_OPTIONS=--tls-min-v1.2,强制版本为tls1.2  node server/app.js 后ok


## 参考：

https://nodejs.org/api/tls.html#tls_modifying_the_default_tls_cipher_suite

https://nodejs.org/api/cli.html#cli_node_options_options

