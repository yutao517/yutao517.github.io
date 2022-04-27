---
layout: article
title: Kubernetes(官方实例)—分配内存资源
tags: Kubernetes
category: blog
date: 2022-04-27 20:30:00 +08:00
mermaid: true
---
[https://kubernetes.io/zh/docs/tasks/configure-pod-container/assign-memory-resource/](https://kubernetes.io/zh/docs/tasks/configure-pod-container/assign-memory-resource/)
## 指定内存请求和限制

创建命名空间

```bash
kubectl create namespace mem-example
```

新建一个限制指定内存请求的 kube.yml文件
```bash
vim memory-request-limit.yaml
```
该容器的内存请求为 50 MiB，内存限制为 100 MiB
```bash
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

```bash
kubectl apply -f memory-request-limit.yaml --namespace=mem-example
#开始创建 Pod
```

```bash
kubectl get pod -n mem-example
#查看Pod 中的容器是否已运行
```

## 超过容器限制的内存
```bash
vim memory-request-limit2.yaml
```
vim memory 容器将会请求 100 MiB 内存，并且内存会被限制在 200 MiB 以内
```bash
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-2
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-2-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
```
在配置文件的 args 部分中，你可以看到容器会尝试分配 250 MiB 内存，这远高于 100 MiB 的限制。
```bash
kubectl apply -f memory-request-limit2.yaml --namespace=mem-example
```

```bash
kubectl get pod -n mem-example
#查看Pod 中的容器是否已运行
```
因为超过资源，所以无法启动。
