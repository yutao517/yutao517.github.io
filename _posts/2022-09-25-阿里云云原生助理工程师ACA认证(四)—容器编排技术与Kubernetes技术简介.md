---
layout: article
title: 阿里云云原生助理工程师ACA认证(四)—容器编排技术与Kubernetes技术简介
tags: 阿里云云原生容器工程师 Kubernetes Docker
category: blog
date: 2022-09-25 15:27:00 +08:00
mermaid: true
---
## 容器编排技术的崛起

**背景**

随着容器技术的快速成熟与推广，容器技术解决了开发人员生产力问题，使DevOps 工作流变得异常流畅。开发人员可以创建容器镜像，运行容器并在该容器中开发代码，再将其部署在本地数据中心或公共云环境中。

越来越多的企业开始使用容器，并把企业自身的生产业务封装进容器，导致容器管理和编排遇到困难。在生产环境中，会涉及到多个容器；这些容器必须跨多个服务器主机进行部署。用户需要容器实施分组管理，以便跨所有容器提供网络、安全、监控等服务。

**容器编排带来的价值**

容器编排：指自动化容器的部署、管理、扩展和联网。可以为需要部署和管理成百上千个容器和主机的企业提供便利。

- 大规模容器自动化部署
- 敏捷高效的资源调度
- 强大的弹性伸缩能力
- 容器服务自愈能力
- 自动化的服务发现


**容器编排的“三国争霸”**

 DockerSwarm、Mesos、Kubernetes都在容器编排领域展开角逐。Swarm 和Mesos的特点，那就是各自只在生态和技术方面比较强。Kubernetes则兼具了两者优势，最终在2017年“三国争霸”的局面中得以胜出，成为了当时直到现在的容器编排标准。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/bc880fa9-a277-4bc7-9f44-d2b94d61c656)


**Kubernetes容器编排的事实标准**

**Kubernetes**

- Google开源的一个容器编排引擎
- 支持自动化部署、大规模可伸缩、应用容器化管理
- 在生产环境中部署一个应用程序时，通常要部署该应用的多个实例以便对应用请求进行负载均衡

**核心功能**

- 服务发现与负载均衡
- 容器的自动装箱
- 存储编排
- 自动化容器恢复
- 自动发布与回滚
- 配置与密文管理
- 批量执行
- 水平伸缩

Kubernetes是一个自动化的容器编排平台，它负责应用的部署、应用的弹性以及应用的管理，这些都是基于容器的。

## Kubernetes核心概念
**Kubernetes核心概念-Pod**

**Pod**：是Kubernetes中能够创建和部署的最小单元；是Kubernetes集群中的一个应用实例，总是部署在同一个节点Node上；包含了一个或多个容器，还包括了存储、网络等各个容器共享的资源；支持多种容器环境，Docker则是最流行的容器环境

- 最小的调度及资源单位
- 由一个或者多个容器组成
- 定义容器的运行方式
- 提供给容器共享的运行环境

**Kubernetes核心概念-Volume**

**Volume**：Pod中能够被多个容器共享的磁盘目录；用来管理Kubernetes存储的；用来声明在Pod中的容器可以访问文件目录的；一个卷可以被挂载在Pod中一个或者多个容器的指定路径下面；声明在Pod中的容器可访问的文件目录；可以被挂载在Pod中一个或者多个容器的路径；支持多种后端存储的抽象:本地存储、分布式存储、阿里云存储等

Docker提供了Volume机制以便实现数据的持久化，Kubernetes中Volume的概念与Docker中的 Volume类似

**Kubernetes核心概念-Deployment**

**Deployment**：是Kubernetes中部署应用最常见的一种方式；做应用的真正的管理；Pod是组成 Deployment最小的单元；定义一组Pod的副本数目、版本等；通过控制器维持Pod数目，自动恢复失败的Pod；通过控制器以指定的策略控制版本

**Kubernetes核心概念-Service**

**service**：是Kubernetes核心的概念；通过创建Service，可以为一组具有相同功能的容器应用提供一个统一的入口地址；将请求进行负载分发到后端的各个容器应用上；提供了一个或者多个Pod实例的稳定访问地址；支持多种访问方式实现:Cluster IP，NodePort，LoadBalance


**Kubernetes核心概念-Namespace**

**Namespace**：用来做一个集群内部的逻辑隔离的，它包括鉴权、资源管理等；每个资源都属于一个Namespace；同一个Namespace中的资源名唯一；不同Namespace中资源可重名

Namespaces可以理解为Kubernetes集群中的虚拟化集群。在一个Kubernetes集群中可以拥有多个命名空间，它们在逻辑上彼此隔离,他们可以为软件开发团队提供组织，安全、性能等方面的保障。

## Kubernetes核心架构

Kubernetes系统用于管理分布式节点集群中的微服务或容器化应用程序；提供了零停机时间部署、自动回滚、缩放和容器的自愈(其中包括自动配置、自动重启、自动复制的高弹性基础设施，以及容器的自动缩放等)功能。

- Kubernetes Master (主控节点)：集群的“大脑”，负责管理所有节点(Node)；负责调度Pod在哪些节点上运行；负责控制集群运行过程中的所有状态
- Kubuernetes Node (工作节点)∶负责管理所有容(Container)；负责监控/上报所有Pod的运行状态

**Kubernetes架构- Master服务端**

- kube-apiserver：集群的HTTP REST API接口，是集群控制的入口
- kube-controller manager：集群中所有资源对象的自动化控制中心
- kube-scheduler：集群中Pod资源对象的调度服务
- Etcd：一个分布式的一个存储系统

**Kubernetes架构-Node客户端组件**

- kubelet组件：负责管理节点上容器的创建、删除、启停等任务,与 Master节点进行通信
- kube-proxy：负责Kubernetes服务的通信及负载均衡服务
- Container Runtime：负责容器的基础管理服务,接收kubelet组件的指令


## Kubernetes流程与调度
**流程**

①发出指令 
②API响应指令，存储etcd，创建Deployment 
③控制器发现资源，并加入到工作队列，创建POD 
④创建完成，更新存储etcd 
⑤资源调度根据规则将POD绑定主机 
⑥绑定结果存储etcd 
⑦Kubelet基于运行情况增加POD 
⑧Kubelet创建增加的新POD 
⑨kube-proxy对POD服务发现和负载均衡 
⑩控制器检测POD运行情况，删除或重新创建


![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/57ba546b-cf4b-4b72-9506-4cfcb6f78225)

**Kubernetes使用场景-调度**

**调度器**：Kubernetes核心组件之一，承载着整个集群资源的调度功能；根据特定调度算法和策略，将Pod调度到最优工作节点上；更合理与充分的利用集群计算资源,使资源更好的服务于业务服务的需求

> 等待调度——过滤
> 正在调度——策略
> 集群状态——运行

**Kubernetes使用场景-自动恢复**

Kubernetes的主要好处之一是它具有管理和维护集群中容器的能力；可以提供服务零停机时间的保障；Pod或容器出现故障时Kubernetes还可以让系统实现”自愈”

**Kubernetes使用场景-弹性伸缩**

Kubernetes 有业务负载检查的能力，会根据业务承担的负载，对业务进行扩容，实现资源弹性伸缩。

## Kubernetes价值及优势
**Kubernetes优势：一个平台搞定所有**

- 使用Kubernetes，部署任何应用都是小菜一碟
- 只要应用可以打包进容器，Kubernetes就一定能启动它
- 不管什么语言什么框架写的应用(Java,Python,Node.js)
- Kubernetes 都可以在任何环境中安全的启动它,物理机、虚拟机、云环境（(公有云、私有云、混合云)

**Kubernetes优势：高效的利用资源**

- Kubernetes可以帮助我们节省开销，高效的利用内存、处理器等资源。Kubernetes 如果发现有节点工作不饱和，便会重新分配 pod。
- 如果一个节点宕机了，Kubernetes 会自动重新创建之前运行在此节点上的pod，在其他节点上运行。

**Kubernetes优势:开箱即用的自动缩放能力**

- 网络、负载均衡、复制等特性，对于Kubernetes都是开箱即用的。
- pod是无状态运行的，任何时候有pod 宕机，立马会有其他pod接替它的工作，用户完全无感。如果用户量突然暴增，现有的- pod规模不足了,那么会自动创建出一批新的 pod，以适应当前的需求。
- 反之亦然，当负载降下来的时候，Kubernetes也会自动缩减pod的数量。

**Kubernetes优势：使CI/CD更加简单**

你不必精通自动化部署和持续集成/部署这类工具，只需要对CI服务写个简单的脚本然后运行它，就会使用你的代码创建一个新的 pod，并部署到Kubernetes集群里面。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/ae7bc5eb-c5ce-424e-82e1-6c5b9adad7b0)


应用打包在容器中使其可以安全的运行在任何地方，例如你的PC、一个云服务器，使得测试极其简单。

**Kubernetes优势:高可用及高可靠**

Kubernetes 如此流行的一个重要原因是:
- 应用会一直顺利运行，不会被pod或节点的故障所中断
- 如果出现故障，Kubernetes会创建必要数量的应用镜像，并分配到健康的pod或节点中，直到系统恢复
- 用户不会感到任何不适。
