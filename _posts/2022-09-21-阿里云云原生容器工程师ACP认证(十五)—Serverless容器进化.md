---
layout: article
title: 阿里云云原生容器工程师ACP认证(十五)—Serverless容器进化
tags: 阿里云云原生容器工程师 Kubernetes Docker
category: blog
date: 2022-09-21 14:02:00 +08:00
mermaid: true
---
## Serverless容器 - 基本概念
**特点**
- 用户无需购买和管理服务器
- 直接部署容器应用
- 提高容器应用部署的敏捷度和弹性能力
- 降低用户计算成本
- 让用户聚焦业务应用

**优势**

- 敏捷部署、安全隔离、生态链接、高移植性
- 无需节点容量规划
- 无需OS和系统软件维护
- 零基础设施运维
- “无限”容量、秒级扩容、基于容器扩容
- 更高的资源利用率、更低计算成本 

## Serverless容器 - 发展趋势
**两种技术趋势的整合**
- 云平台托管的后端服务 BaaS
- 无状态计算模型：函数服务 FaaS

**Gartner**
- 到2023年，70％AI任务会通过容器、Serverless等计算模型构建

**AWS**
- 在2019年40％的ECS新客户采用Serverless Container

**采纳 Serverless技术的行业广泛**

- 外包经济、金融业、服务业

**Serverless 是云计算必经的一场革命**

**Serverless 会越来越流行**

![在这里插入图片描述](https://img-blog.csdnimg.cn/782f916628dc4b24a6abd2667707201b.png)

Gartner在2020年 Public Cloud Container Service Market评估报告中把Serverless容器作为云厂商容器服务平台的主要差异化之一
  - 其中将产品能力划分为Serverless 容器实例和Serverless Kubernetes两类
  - 这与阿里云ECI／ASK的产品定位高度一致
  
Gartner报告中也谈到Serverless容器业界标准未定，云厂商有很多空间通过技术创新提供独特的增值能力，其对云厂商的建议是：
  - 扩展Serverless容器应用场景和产品组合，迁移更多普通容器workload到serverless容器服务。
  - 推进Serverless容器的标准化，减轻用户对云厂商锁定的担忧。

## Serverless容器 - 构架思考
- 在阿里云Kubernetes服务中使用Serverless容器的场景，我们同时支持在K8s集群中使用ECI的方案ACK on ECI，和针对Serverless Kubernetes的极致优化的产品ASK。二者可以实现互补，覆盖满足不同客户的的诉求。
- 不同于标准K8s，Serverless K8s与laaS基础设施深度整合，其产品模式更利于公有云厂商通过技术创新，提升规模、效率和能力。
- 在架构层面我们将Serverless容器分成容器编排和计算资源池两层，下面我们将对这两层进行深度剖析，分享我们对Serverless容器架构和产品背后的关键思考。

## Serverless容器-构架思考：容器编排

Kubernetes在容器编排的成功不止得益于Google的光环和CNCF（云原生计算基金会）的努力运作。背后是其在Google Borg大规模分布式资源调度和自动化运维领域的沉淀和升华。
其中几个技术要点：
- 声明式API
- 可扩展性架构
- 可移植性

## Serverless容器- 构架思考：设计原则

Serverless Kubernetes 必须要能兼容Kubernetes生态，提供K8s的核心价值，此外要能和云的能力深度整合。
- 用户可以直接使用Kubernetes的声明式API，兼容Kubernetes的应用定义，Deployment，StatefulSet，Job，Service等无需修改。
- 全兼容Kubernetes的扩展机制，这个很重要，这样才能让Serverless Kubernetes支持更多的工作负载。此外Serverless K8s自身的组件也是严格遵守K8s的状态逼近的控制模式。
- Kubernetes的能力尽可能充分利用云的能力来实现，比如资源的调度、负载均衡、服务发现等。根本性简化容器平台的设计，提升规模，降低用户运维复杂性。
- 这些实现应该是对用户透明的，保障可移植性，让用户现有应用可以平滑部署在Serverless K8s之上，也应该允许用户应用混合部署在传统容器和Serverless容器之上。

## Serverless容器 - 构架思考：Nodeless
传统的Kubernetes采用以节点为中心的架构设计：
- 节点是pod的运行载体，Kubernetes调度器在工作节点池中选择合适的node来运行pod，并利用Kubelet完成对Pod进行生命周期管理和自动化运维
- 当节点池资源不够时，需要对节点池进行扩容，再对容器化应用进行扩容。

对于Serverless Kubernetes而言：

- 最重要的一个概念是将容器的运行时和具体的节点运行环境解耦。
- 用户无需关注node运维和安全，降低运维成本
- 极大简化了容器弹性实现，无需按照容量规划，按需创建容器应用Pod即可
- 此外，Serverless容器运行时可以被整个云弹性计算基础设施所支撑，保障整体弹性的成本和规模。

## Serverless容器 - 构架思考：Viking

在现有Kubernetes的设计实现上，进行了扩展和优化。构建了Cloud Scale的Nodeless K8s架构 - 内部的产品代号为Viking，因为古代维京战船以迅捷和便于操作而著称。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a61b1ed979af47cfafd54f653f14a845.png)

## Serverless容器 - 构架思考：优化方向

**Scheduler**

传统 K8s scheduler 的主要功能是从一批节点中选择一个合适的 node 来调度 Pod，满足资源、亲和性等多种约束。由于在 Serverless K8s 场景中没有 node 的概念，资源只受限于底层弹性计算库存，我们只需要保留一些基本的 AZ 亲和性等概念支持即可。这样 scheduler 的工作被极大简化，执行效率极大提升。此外我们定制扩展了 scheduler，可以对 Serverless workload 进行更多的编排优化，可以在保证应用可用性的前提下充分降低了计算成本。


**可伸缩性**

K8s 的可伸缩性受到众多因素的影响，其中一个就是节点数量。为了保障 Kubernetes 兼容性，AWS EKS on Fargate 采用 Pod 和 Node 1:1 模型（一个虚拟节点运行一个 Pod），这样将严重限制了集群的可扩展性，目前单集群最多支持 1000 个 Pod。我们认为，这样的选择无法满足大规模应用场景的需求。在 ASK 中我们在保持了 Kubernetes 兼容性的同时，解决了集群规模受限于 Node 影响，单集群可以轻松支持 10K Pod。此外传统 K8s 集群中还有很多因素会影响集群的可伸缩性，比如部署在节点上的 kube-proxy，在支持 clusterIP 时，任何单个 endpoint 变更时就会引起全集群的变更风暴。在这些地方 Serverless K8s 也使用了一些创新的方法限制变更传播的范围，这些领域我们将持续优化。

**基于云的控制器实现**

我们基于阿里云的云服务实现了 kube-proxy、CoreDNS、Ingress Controller 的行为，降低系统复杂性，比如：

- 利用阿里云的 DNS 服务 PrivateZone，为 ECI 实例动态配置 DNS 地址解析，支持了 headless service
- 通过 SLB 提供了提供负载均衡能力
- 通过 SLB/ALB 提供的 7 层路由来实现 Ingress 的路由规则

**面向工作负载的深度优化**

未来充分发挥 Serverless 容器的能力，我们需要针对工作负载的特性进行深度优化。

- **Knative**：Knative 是 Kubernetes 生态下一种 Serverless 应用框架，其中 serving 模块支持根据流量自动扩缩容和缩容到 0 的能力。基于 Serverless K8s 能力，阿里云 Knative 可以提供一些新的差异化功能，比如支持自动缩容到最低成本 ECI 实例规格，这样可以在保障冷启动时间的 SLA 并有效降低计算成本。此外通过 SLB/ALB 实现了 Ingress Gateway，有效地降低了系统复杂性并降低成本。

- 在 Spark 等大规模计算任务场景下，也通过一些**垂直优化**手段提高大批量任务的创建效率。这些能力已经在江苏跨域的用户场景中得到验证。

