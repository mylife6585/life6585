---
title: "Weichat Callback Failure"
date: 2022-09-07T16:28:52+08:00
description: "istio环境微信回调失败"
tags: ["istio","wechat"]
categories: ["devops"]
keywords: ["istio","wechat"]
draft: false
isCJKLanguage: true
---
## 现象

微信回调接口失败，curl、postman 正常


## 分析

编辑istio configmap显示ingress-gateway 日志

`kubectl edit configmap -n istio-system`

增加
```
data:
  mesh: |-
    accessLogEncoding: JSON
    accessLogFile: /dev/stdout
```

ingress-gateway 日志

```
{
  "downstream_remote_address":"172.23.249.64:12982",
  "route_name":null,
  "bytes_received":0,
  "upstream_local_address":null,
  "bytes_sent":0,
  "duration":0,
  "response_flags":"-",
  "protocol":"HTTP/1.0",
  "downstream_local_address":"172.23.141.49:8443",
  "upstream_service_time":null,
  "upstream_host":null,
  "request_id":null,
  "x_forwarded_for":null,
  "requested_server_name":"base-app.creams.io",
  "start_time":"2022-09-07T07:38:52.549Z",
  "authority":"base-app.creams.io",
  "path":"/api/wechat-bridge/event/callback?signature=89e1e32c0d51d3834754a70b67fd878354724e93&echostr=5512910522763997071&timestamp=1662536332&nonce=790690840",
  "response_code":426,
  "upstream_transport_failure_reason":null,
  "method":"GET",
  "upstream_cluster":null,
  "connection_termination_details":null,
  "user_agent":"Mozilla/4.0",
  "response_code_details":"low_version"
}

```

通过日志可以知道是因为微信回调的时候使用HTTP1.0 ，istio默认不支持导致


## 解决

通过环境变量，设置istio envoy 支持http1.0

```
kubectl edit deploy -n istio-system 
```

增加环境变量 PILOT_HTTP10 为 1。



## 参考

https://xie.infoq.cn/article/9fd56ee9a0a8f177394d9b9d9

https://makeoptim.com/istio-faq/istio-support-http10#:~:text=Istio%20%E9%BB%98%E8%AE%A4%E5%8F%AA%E6%94%AF%E6%8C%81HTTP,%E5%B9%B6%E4%B8%8D%E6%94%AF%E6%8C%81HTTP%2F1.0%E3%80%82


