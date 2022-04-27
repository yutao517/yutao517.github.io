---
layout: article
title: Kubernetes(五)—Pod启动过程和生命周期
tags: Kubernetes
category: blog
date: 2022-04-27 22:00:00 +08:00
mermaid: true
---
## 整体流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/75e2e52da941477e98913432ad5035e4.png)


- 用户通过kubectl(或者其他API客户端)提交Pod创建指令，指令可以是命令也可以是yaml文件。Pod创建指令信息传给API Server， API Server再将Pod信息存入ETCD。
- Master组件中的Controller Manager得到API Server的Pod创建指令信息，做编排工作，创建应用所需要的Pod，并将创建信息反馈给API Server，API Server再将Pod信息更新存储到ETCD。
- Scheduler通过使用“watch”监控到API Server中新Pod的变化，根据调度算法，为Pod分配一个节点Node，并将分配结果反馈给API Server，API Server再将Pod信息更新存储到ETCD。
- API Server通知对应节点的kubelet，kubelet发现Pod调度到本节点，创建并运行Pod的容器。
- Kube-Proxy给pod分配网络资源，将Pod的网络和K8S集群的网络连通，发布Pod对应的服务。

## Pod状态

![在这里插入图片描述](https://img-blog.csdnimg.cn/9e6639d7c27e48559df3c0f5fbab7e09.png)

详细状态

![在这里插入图片描述](https://img-blog.csdnimg.cn/003ab3bb40db4bcf8bbdbed65dd203ee.png)

## 容器状态

容器的状态有三种：Waiting（等待）、Running（运行中）和 Terminated（已终止）。
要检查Pod中容器的状态，使用 `kubectl describe pod <pod 名称>`命令
