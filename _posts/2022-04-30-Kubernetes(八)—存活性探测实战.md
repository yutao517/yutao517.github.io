---
layout: article
title: Kubernetes(八)—存活性探测实战
tags: Kubernetes
category: blog
date: 2022-04-30 20:32:00 +08:00
mermaid: true
---
## 存活探针案例
**EXEC探针**

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
30s之后删除healthy文件，初始化后5s探针开始探/tmp/healthy文件，周期为5s，30s之后删除healthy文件报错，重启。
```bash
kubectl apply -f liveness.yaml 
```

```bash
kubectl describe pod liveness-exec
```

**HTTP探针**

```bash
apiVersion: v1
kind: Pod
metadata:
  name: http-probe
  namespace: default
  labels:
    probe: http
spec:
  containers:
  - name: http-probe
    image: nginx:1.12
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    lifecycle:
      postStart:
        exec:
          command: ["/bin/bash","-c","echo Http-Probe > /usr/share/nginx/html/ishealth.html"]
    livenessProbe:
      httpGet:
        path: /ishealth.html
        port: http
        scheme: HTTP
```
进入容器

```bash
kubectl exec -it http-probe -- /bin/bash
```
删除文件
```bash
rm -rf /usr/share/nginx/html/ishealth.html
```
查看pod信息

```bash
kubectl get pod 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/9506969fb90e412ba29ca0be2739b65f.png)

重启次数由0变为1

**TCP探针实战**

```bash
apiVersion: v1
kind: Pod
metadata:
  name: tcp-probe
  namespace: default
  labels:
    probe: tcp
spec:
  containers:
  - name: tcp-rpobe
    image: nginx:1.12
    ports:
    - name: http
      containerPort: 80
    livenessProbe:
      tcpSocket:
        port: http
```
```bash
kubectl apply -f tcp-probe.yaml
```

```bash
kubectl describe pod tcp-probe
```
