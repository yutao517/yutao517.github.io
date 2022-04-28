---
layout: article
title: Kubernetes(六)—配置Pod使用数据卷进行存储
tags: Kubernetes
category: blog
date: 2022-04-28 16:00:00 +08:00
mermaid: true
---
Kubernetes提供了众多的volume类型，包括emptyDir、hostPath、nfs、glusterfs、cephfs、ceph rbd等。具体可以参考[官方文档](https://kubernetes.io/docs/concepts/storage/volumes/)。

[本文参考文档](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-volume-storage/)

## emptyDir
emptyDir类型的volume在pod分配到node上时被创建，kubernetes会在node上自动分配 一个目录，因此无需指定宿主机node上对应的目录文件。这个目录的初始内容为空，当Pod从node上移除时，emptyDir中的数据会被永久删除。emptyDir Volume主要用于某些应用程序无需永久保存的临时目录，多个容器的共享目录等。

**配置yaml启动指令**

```bash
vim redis.yaml 
```

```bash
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```
**启动pod**
```bash
kubectl apply -f redis.yaml
#创建 Pod
```
```bash
kubectl get pod redis --watch
#验证 Pod 中的容器是否正在运行，并留意更改
```
**终端**

复制会话在另一个终端，用xshell 连接manager。
```bash
kubectl exec -it redis -- /bin/bash
#进入终端
```
```bash
cd /data/redis/
echo Hello > test-file
```

安装ps命令
```bash
apt-get update
apt-get install procps
ps aux
```

结束 Redis 进程

```bash
 kill <pid>
```
容器终止并重新启动。因为Redis Pod的restartPolicy 为 Always

![在这里插入图片描述](https://img-blog.csdnimg.cn/544f1826de00459b80201c1839e54eba.png)
再次进入终端
```bash
kubectl exec -it redis -- /bin/bash
cd /data/redis/
ls
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3fa193c4cb264a8e953e509c49f81f83.png)
可以看到重启的容器数据卷仍然在，但是ps命令无法使用，说明这是一个新的容器， 在容器终端的/data/redis目录下挂载了数据卷。

## hostPath
hostPath Volume为pod挂载宿主机上的目录或文件，使得容器可以使用宿主机的高速文件系统进行存储。缺点是，在k8s中，pod都是动态在各node节点上调度。当一个pod在当前node节点上启动并通过hostPath存储了文件到本地以后，下次调度到另一个节点上启动时，就无法使用在之前节点上存储的文件。

**创建yaml指令**
```bash
apiVersion: v1
kind: Pod
metadata:
  name: redis-wangyutao
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage-wangyutao
      mountPath: /data/redis
  volumes:
  - name: redis-storage-wangyutao
    hostPath:
      path: /wangyutao
```
**启动pod**
```bash
kubectl apply -f redis.yaml
#启动
kubectl get pod -o wide
#查看调度情况
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/328f77bc0b814fb49cbff93f5ef1728b.png)

这里调度到了node1，所以进入node1查看挂载目录

![在这里插入图片描述](https://img-blog.csdnimg.cn/3b1118b4d4cc4135a72711bf2a41f524.png)

已经创建该挂载目录，使用docker命令查看容器编号

```bash
docker ps|grep redis
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2026875b992846e7bf77188534a5db23.png)

进入容器
```bash
docker exec -it b0d8c14551f9  /bin/bash
```

执行如下操作，可以看到宿主机的挂载目录下的内容有所变化


![在这里插入图片描述](https://img-blog.csdnimg.cn/adac69d1fbb7486dbb52b175864f1d28.png)
