---
title: "请求token为null"
date: 2021-11-25T09:45:03+08:00
description: "刷新页面之后token丢失"
tags: ["oauth","token"]
categories: ["devops"]
keywords: ["oauth","token"]
draft: false
isCJKLanguage: true
---
## 现象

刷新页面，退出到登录页面,这种情况不定时出现

local storage中到token信息被清理

failed to load response data: no data found for resource with given identifier


## 原因及解决

错误设置access token 的时间大于 refresh token 时间，

导致前端认为现在需要清空所有token，重新获取

但获取的还是老的access token, refresh token 已被清理

刷新页面，又退出到登录页面



