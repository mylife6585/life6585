---
title: "Istio 1.6安装问题记录"
date: 2020-06-20T22:22:22+08:00
description: "Istio 1.6安装问题记录"
tags: ["istio"]
categories: ["virtualization"]
keywords: ["istio"]
draft: false
isCJKLanguage: true
---


## problem

k8s 1.17二进制部署安装，安装istio 1.6.0的时候碰到以下报错

> configmap "istio-ca-root-cert" not found

> MountVolume.SetUp failed for volume "istio-token" : failed to fetch token: the API server does not have TokenRequest endpoints enabled

> 2020-06-10T13:51:44.017157Z     warn    serverca        Authentication failed: Authenticator ClientCertAuthenticator at index 0 got error: no verified chain is found. Authenticator KubeJWTAuthenticator at index 1 got error: failed to validate the JWT: the service account authentication returns an error: [invalid bearer token, square/go-jose: error in cryptographic primitive]. Authenticator ClientCertAuthenticator at index 2 got error: no verified chain is found.
> 2020-06-10T13:51:44.017202Z     warn    serverca        request authentication failure
> 2020-06-10T13:51:44.122245Z     info    grpc: Server.Serve failed to complete security handshake from "172.23.0.231:51822": EOF
> 2020-06-10T13:51:44.821641Z     info    grpc: Server.Serve failed to complete security handshake from "172.23.144.6:49404": remote error: tls: error decrypting message


同一个问题引起: k8s api server缺少service account相关配置或者配置错误

正确配置(证书路径根据实际情况配置)：

```
  --service-account-signing-key-file=/etc/kubernetes/certs/kubernetes-key.pem \
  --service-account-key-file=/etc/kubernetes/certs/kubernetes.pem \
  --service-account-issuer=api \
  --service-account-api-audiences=api,vault,factors \
```

重启k8s api server

## 安装istio


```bash
# 安装istio
wget https://github.com/istio/istio/releases/download/1.6.1/istio-1.6.1-linux-amd64.tar.gz

tar -zxvf istio-1.6.1-linux-amd64.tar.gz

cd istio-1.6.1

cp bin/istioctl /usr/local/bin/

cp tools/istioctl.bash ~/

~/.bashrc 增加 source ~/istioctl.bash

source ~/.bashrc

istioctl manifest versions

istioctl manifest generate --set profile=default \
         --set values.gateways.istio-ingressgateway.type=NodePort \
         --set components.cni.enabled=true \
         --set values.cni.cniBinDir='/opt/cni/bin' \
         --set values.cni.cniConfDir='/etc/cni/net.d' \
         --set components.cni.namespace=kube-system > generated-manifest.yaml

kubectl apply -f generated-manifest.yaml

istioctl verify-install -f ./generated-manifest.yaml

```

```bash
# 卸载istio
istioctl manifest generate --set profile=default | kubectl delete -f -
```

## reference

https://www.cnblogs.com/charlieroro/p/12857372.html


https://mp.weixin.qq.com/s/DrvBEVpMMt1pJBKPbyPcyQ


