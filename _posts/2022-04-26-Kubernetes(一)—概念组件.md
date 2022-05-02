---
layout: article
title: Kubernetes(一)—概念组件
tags: Kubernetes
category: blog
date: 2022-04-26 15:35:00 +08:00
mermaid: true
---
## 简介
**Kubernetes**（常简称为**K8s**）是用于自动部署，扩展和管理容器化应用程序的开源系统。该系统由Google设计并捐赠给Cloud Native Computing Foundation（今属Linux基金会）来使用。它旨在提供“跨主机集群的自动部署、扩展以及运行应用程序容器的平台”。它支持一系列容器工具，包括Docker。

Docker 提供容器的⽣命周期管理和 Docker 镜像构建运⾏时容器。但是，由于这些单独的容器有时必须跨主机通信，这时我们需要使⽤ Kubernetes 来解决这个问题。因此，我们说 Docker 构建容器，但这些容器通过 Kubernetes 来进⾏跨主机相互通信。我们还可以使⽤ Kubernetes 手动关联和编排在多个主机上运⾏的容器。

**功能用途**
- 自动化部署，扩展和管理容器。
- 组成容器集群。
- 节省资源，优化硬件资源的使用
- 容器弹性，如果容器失效立即替换

**特点**
- 可移植 : 支持公有云,私有云,混合云,多重云
- 可扩展 : 模块化,插件化,可挂载,可组合,支持各种形式的扩展
- 自动化 : 自动部署,自动重启,自动复制,自动伸缩/扩展,通过声明式语法提供了强大的自修复能力

## 组件

### 控制平面组件（Control Plane Components）：
控制平面的组件也就是Master组件对集群做出全局决策(比如调度)，以及检测和响应集群事件（例如，当不满足部署的 replicas 字段时，启动新的控制平面组件可以在集群中的任何节点上运行）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/28cb4e9dc1d54de39c6684732d490123.png)

**API Server** 
 
 - **API Server**是 Kubernetes 控制面的**前端**，是整个Kubernetes集群的**网关**，以及接收、校验并响应所有的REST请求，结果状态被永久存储在ETCD中。

**ETCD**
 
 - **ETCD**是兼具一致性和高可用性的**键值数据库**，以键值对的方式持久化存储Kubernetes集群中的状态信息。

**Scheduler**
  
 - **调度器**，负责工作与Kubernetes集群的底层，会根据Kubernetes集群中各个节点的状态以及对容器的资源需求进行调度决策,选择节点让 Pod 在上面运行。Kubernetes也支持用户自定义调度器。Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元，，这些容器共享存储、网络，就Docker 概念的术语而言，Pod 类似于共享名字空间和文件系统卷的一组 Docker 容器

**kube controller manager**
 
 **集群控制器**，运行控制器进程的控制平面组件，可以完成大多数集群级别的功能。从逻辑上讲，每个控制器都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在一个进程中运行。
 
   这些控制器包括:
 ，这些容器共享存储、网络，
 - **节点控制器**（Node Controller）: 负责在节点出现故障时进行通知和响应；
 
 - **任务控制器**（Job controller）: 监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成；
 
 - **端点控制器**（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)；
 
 - **服务帐户和令牌控制器**（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌
 
**cloud controller manager**

**云控制器管理器**是指嵌入特定云的控制逻辑的 控制平面组件。 云控制器管理器使得你可以将你的集群连接到云提供商的 API 之上， 并将与该云平台交互的组件同与你的集群交互的组件分离开来。

- **节点（Node）控制器**：用于在节点终止响应后检查云提供商以确定节点是否已被删除

- **路由（Route）控制器**：用于在底层云基础架构中设置路由

- **Service控制器**：用于创建、更新和删除云提供商负载均衡器


### Node 组件

Kubelet

- **Kubelet**是运行在Kubernetes集群中Node结点上运行的**代理**，是一个守护进程， 它保证容器（containers）都运行在 Pod 中。

Kube-proxy

- **Kube-Proxy** 是集群中每个节点上运行的网络代理，负责为Service对象生成iptables或者是ipvs规则，从而捕获访问该Service的数据流量，并将这些流量转发给后端的Pod对象，是实现 Kubernetes 服务（Service） 概念的一部分。
