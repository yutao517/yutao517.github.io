---
layout: article
title: Kubernetes(十四)—PV和PVC.md
tags: Kubernetes
category: blog
date: 2022-05-05 19:58:15 +08:00
mermaid: true
---
## 简介

**PersistentVolume (PV)** 由管理员创建和维护，是外部存储系统中的一块存储空间，是Volume 之类的卷插件，PV 也是集群中的资源。PV 具有持久性，独立于使用 PV 的 Pod 的生命周期（pod被删除了，我们的PV依然会被保留，类似于卷）与 Volume 一样。

**访问模式**
- ReadWriteOnce（RWO）：读写权限，但是只能被单个节点挂载
- ReadOnlyMany（ROX）：只读权限，可以被多个节点挂载
- ReadWriteMany（RWX）：读写权限，可以被多个节点挂载

**PersistentVolumeClaim (PVC)** 持久化卷声明，是对 PV 的申请，PVC 通常由普通用户创建和维护。PVC 和 Pod 比较类似，Pod 消耗的是节点，PVC 消耗的是 PV 资源，Pod 可以请求 CPU 和内存，而 PVC 可以请求特定的存储空间和访问模式（比如只读）等信息，Kubernetes 会查找并提供满足条件的 PV。

## 使用
[参考文档](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

**前提**

在每个node节点创建一个/mnt/data目录存放挂载数据
 index.html
 
```bash
mkdir -p /mnt/data
cat <<EOF >/mnt/data/index.html
Hello Kubernetes
EOF
```
## 创建PV
创建一个 hostPath 类型的 PersistentVolume，但是在生产集群中，一般不会使用 hostPath。 集群管理员会提供网络存储资源，比如 Google Compute Engine 持久盘卷、NFS 共享卷或 Amazon Elastic Block Store 卷
```bash
cat<<EOF>pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
EOF
```

```bash
kubectl apply -f pv.yaml 
```
```bash
kubectl get pv
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d0c4bce633794babb26a3f9610627a68.png)

STATUS为Avaliable，因为还没有绑定到PVC

## 创建PVC

```bash
cat<<EOF>pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi 
      #定义要申请的空间大小
EOF
```

```bash
kubectl apply -f pvc.yaml 
```
```bash
kubectl get pv
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/82467552d99149a8898dce67e19f4b0e.png)

状态已绑定

## 创建 Pod

```bash
cat<<EOF>pv-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
  labels:
    run: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
---
apiVersion: v1
kind: Service
metadata:
  name: task-pv-pod
  labels:
    run: task-pv-pod
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    run: task-pv-pod
EOF
```

```bash
kubectl apply -f pv-nginx.yaml 
```

```bash
kubectl get pod
kubectl get svc
```
看到服务已经启动

![在这里插入图片描述](https://img-blog.csdnimg.cn/c459218f5a37489e908933a4b7bb5c47.png)

## 验证

浏览器访问可以看到我们挂载的数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/3bdc3de0290f44b79a8d7b3dc59ebd60.png)

进入调度的node1服务器，修改/mnt/data/index.html内容

```bash
 echo 'wangyutao'>/mnt/data/index.html
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e83fdde8b62c441aa66251b04d860dfc.png)

## 清理

```bash
kubectl delete pod task-pv-pod
kubectl delete pvc task-pv-claim
kubectl delete pv task-pv-volume
```
