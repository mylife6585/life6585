---
title: "Swagger Ui Basic Auth"
date: 2022-07-27T10:44:21+08:00
description: "Swagger ui增加认证"
tags: ["swaggerui","k8s"]
categories: ["devops"]
keywords: ["swaggerui","k8s"]
draft: false
isCJKLanguage: true
---

## 创建password文件
```
htpasswd -c /tmp/password username
```


## 修改deployment.yaml ，通过lifecycle增加认证

```
        lifecycle:
          postStart:
            exec:
              command: ["/bin/sh", "-c", 'echo username:password > /etc/nginx/.htpasswd;sed -i "/sendfile/a\  auth_basic_user_file /etc/nginx/.htpasswd;" /etc/nginx/nginx.conf;sed -i "/sendfile/a\  auth_basic \"Restricted Content\";" /etc/nginx/nginx.conf']
```

 username:password 替换成真实数据，密码中如果包含$等特殊字符，需要\ 转义

