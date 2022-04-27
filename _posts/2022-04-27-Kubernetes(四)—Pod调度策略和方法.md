---
layout: article
title: Kubernetes(四)—Pod调度策略和方法
tags: Kubernetes
category: blog
date: 2022-04-27 17:15:00 +08:00
mermaid: true
---

## Pod调度策略和方法
### 全自动调度（Deployment）
根据系统内置调度算法，自动部署容器的多个副本，保持集群中始终运行着指定个数的Pod
###  定向调度（NodeSelector）
内部系统通过一系列算法最终计算出最佳的目标节点。如果需要将Pod调度到指定Node上，则可以通过Node的标签（Label）和Pod的nodeSelector属性相匹配来达到目的

### 亲和性（Affinity）与非亲和性（anti-affinity）

**节点亲和性**(nodeAffinity)：主要是用来控制 Pod 要部署在哪些节点上，以及不能部署在哪些节点上的。

**Pod 亲和性**(podAffinity)：主要解决 Pod 可以和哪些 Pod 部署在同一个拓扑域中的问题。

**Pod 反亲和性**(podAntiAffinity)：反着来的，比如一个节点上运行了某个 Pod，那么我们的模板 Pod 则不希望被调度到这个节点上面去了。

调度可以分成**软策略**和**硬策略**两种方式。所以亲和性又分为硬亲和与软亲和。

- 软策略就是如果你没有满足调度要求的节点的话，POD 就会忽略这条规则，继续完成调度过程
- 硬策略就比较强硬，表示pod必须部署到满足条件的节点上，如果没有满足条件的节点，就不停重试。 

### 污点与容忍
对于 nodeAffinity 无论是硬策略还是软策略方式，都是调度 Pod 到预期节点上。

**污点**（Taints）恰好与之相反，如果一个节点标记为 Taints ，除非 Pod 也被标识为可以**容忍**污点节点，否则该 Taints 节点不会被调度Pod。也就是说如果希望将某些Pod调度到设置有污点的节点上，则需要在Pod的Spec中设置容忍度。
