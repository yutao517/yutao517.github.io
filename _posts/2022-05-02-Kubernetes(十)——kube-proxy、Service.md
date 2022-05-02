---
layout: article
title: Kubernetes(十)——kube-proxy、Service
tags: Kubernetes
category: blog
date: 2022-05-02 14:50:00 +08:00
mermaid: true
---
## 背景
pod 是经常变化的，每次更新 ip 地址都可能会发生变化，如果直接访问容器 ip 的话，会有很大的问题。而且进行扩展的时候，rc 中会有新的 pod 创建出来，出现新的 ip 地址，我们需要一种更灵活的方式来访问 pod 的服务。

## Service简介

针对这个问题，kubernetes 的解决方案是“**服务”（service）** Service是k8s中资源的一种，使k8s实现减少运维工作量，每个服务都一个固定的虚拟 ip，自动并且动态地绑定后面的 pod，所有的网络请求直接访问服务 ip，服务会自动向后端做转发。Service 除了**提供稳定的对外访问方式**之外，还能起到**负载均衡**（Load Balance）的功能，自动把请求流量分布到后端所有的服务上，服务可以做到对客户透明地进行**水平扩展**（scale）。水平扩展就是集群，横向扩展，增加服务器实例，分散负载。

## kube-proxy简介
**Kube-Proxy** 是集群中每个节点上运行的网络代理，负责为Service对象生成iptables或者是ipvs规则，从而捕获访问该Service的数据流量，并将这些流量转发给后端的Pod对象，是实现 **service** 这一功能的关键。

## kube-proxy代理模式
- userspace
- iptables
- lvs(ipvs)
## Service类型
尽管每个 Pod 都有一个唯一的 IP 地址，但是如果没有 Service ，这些 IP 不会暴露在集群外部。

- ClusterIP：在集群的内部 IP 上公开 Service 。这种类型使得 Service 只能从集群内访问。
- NodePort：使用 NAT 在集群中每个选定 Node 的相同端口上公开 Service
- LoadBalancer： 在当前云中创建一个外部负载均衡器(如果支持的话)，并为 Service 分配一个固定的外部IP
- ExternalName： 通过返回带有该名称的 CNAME 记录，使用任意名

**无状态服务(deployment)**

例如nginx

- 认为所有pod都是一样的，不具备与其他实例有不同的关系。
- 没有顺序的要求。
- 不用考虑在哪个Node运行。
- 随意扩容缩容。

**有状态服务(SatefulSet)**

例如mysql

-  集群节点之间的关系。
- 数据不完全一致。
- 实例之间不对等的关系。
- 依靠外部存储的应用。
- 通过dns维持身份

