---
title: "Velero_with_minio"
date: 2023-04-21T11:25:10+08:00
description: "velero结合minio备份、还原k8s"
tags: ["velero","minio"]
categories: ["devops"]
keywords: ["velero","minio"]
draft: false
isCJKLanguage: true
---


## 一、minio

部署minio

```
---
apiVersion: v1
kind: Namespace
metadata:
  name: velero

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.beta.kubernetes.io/storage-provisioner: nas.csi.volcengine.com
  name: velero-storage
  namespace: velero
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  storageClassName: nas-sc

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    volume.beta.kubernetes.io/storage-provisioner: nas.csi.volcengine.com
  name: velero-config
  namespace: velero
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: nas-sc
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: velero
  name: minio
  labels:
    component: minio
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      component: minio
  template:
    metadata:
      labels:
        component: minio
    spec:
      volumes:
      - name: velero-storage
        persistentVolumeClaim:
          claimName: velero-storage
      - name: velero-config
        persistentVolumeClaim:
          claimName: velero-config
      containers:
      - name: minio
        image: minio/minio:latest
        imagePullPolicy: IfNotPresent
        command:
        - /bin/bash
        - -c
        args:
        - minio server /storage --config-dir=/config --address :9000 --console-address :9001
        env:
        - name: MINIO_ROOT_USER
          value: "minio"
        - name: MINIO_ROOT_PASSWORD
          value: "Creams_minio"
        ports:
        - containerPort: 9000
        volumeMounts:
        - name: velero-storage
          mountPath: "/storage"
        - name: velero-config
          mountPath: "/config"

---
apiVersion: v1
kind: Service
metadata:
  namespace: velero
  name: minio
  labels:
    component: minio
spec:
  type: ClusterIP
  ports:
    - port: 9000
      targetPort: 9000
      protocol: TCP
      name: api
    - port: 9001
      targetPort: 9001
      protocol: TCP
      name: console
  selector:
    component: minio

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minio-ingress
  namespace: velero
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: 10240m
spec:
  ingressClassName: nginx
  rules:
  - host: minio.souban.io
    http:
      paths:
      - backend:
          service:
            name: minio
            port:
              number: 9000
        pathType: ImplementationSpecific
  tls:
  - hosts:
    - minio.souban.io
    secretName: souban.io

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minio-console-ingress
  namespace: velero
spec:
  ingressClassName: nginx
  rules:
  - host: minio-console.souban.io
    http:
      paths:
      - backend:
          service:
            name: minio
            port:
              number: 9001
        pathType: ImplementationSpecific
  tls:
  - hosts:
    - minio-console.souban.io
    secretName: souban.io

```

  手动创建velero bucket


## 二、velero

版本：1.10.2

```
#credentials-velero
[default]
aws_access_key_id=minio
aws_secret_access_key=minio
```



```
velero install  --provider aws  --plugins velero/velero-plugin-for-aws:v1.6.1  --bucket velero  --secret-file ./credentials-velero --use-node-agent   --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=https://minio.souban.io --snapshot-location-config region=minio  --namespace=velero --wait
```

```
velero backup-location get
```


## 三、测试

```
velero backup create mysql-backup-1 --include-namespaces nginx-example --default-volumes-to-fs-backup --snapshot-volumes --ttl 87600h
velero backup get
velero backup logs nginx-example-backup-1
velero backup describe nginx-example-backup-1
```

```
velero restore create --from-backup nginx-example-backup-1
velero restore get

```

## 四、problem

1. 备份pvc

使用fs-backup替代snapshot backup



2. 设置过期时间

设置--ttl 参数，默认30天

3. 迁移
4. 不能访问日志

minio.cluster.svc 不能被外部访问导致



## 五、reference

https://velero.io/docs/v1.11/

https://bp.aliyun.com/detail/177?spm=5176.21213303.J_6704733920.19.987653c9hcgFee



