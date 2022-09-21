---
layout: article
title: 阿里云云原生容器工程师ACP认证(十七)—阿里云Serverless容器(ASK)适用场景与核心功能
tags: 阿里云云原生容器工程师 Kubernetes Docker
category: blog
date: 2022-09-21 14:33:00 +08:00
mermaid: true
---
## ASK适用场景

![在这里插入图片描述](https://img-blog.csdnimg.cn/cebb59ff07b84b7388ff78c9f3a1ed8e.png)

## 虚拟节点
ASK集群中基于虚拟节点创建Pod。虚拟节点实现了Kubernetes与弹性容器实例ECI的无缝连接，让Kubernetes集群获得极大的弹性能力，而不必关心底层计算资源容量。

![在这里插入图片描述](https://img-blog.csdnimg.cn/17c0ed58cb1740c5953af195f7ed33f6.png)

在集群中用户可以查看虚拟节点信息，但无需对虚拟节点进行任何操作，虚拟节点也不占用任何计算资源。

## Pod安全隔离

ASK集群中的Pod基于阿里云弹性容器实例ECI运行在安全隔离的容器运行环境中。每个容器实例底层通过轻量级虚拟化安全沙箱技术完全强隔离，容器实例间互不影响。

ECI的容器实例底层运行在Alibaba Cloud Linux2操作系统之上，ASK集群作为Serverless容器服务，您无法访问ECI底层OS运行环境。

## Pod配置

Pod基于弹性容器实例ECI创建，其支持原生的Kubernetes Pod功能，包括启动多个容器、设置环境变量、设置RestartPolicy、设置健康检查命令和挂载volumes、preStop等。同时支持执行命令kubectl logs访问容器日志和执行kubectl exec进入容器。



## 应用负载管理

- 支持Deployment、StatefulSet、Job/CronJob、Pod、CRD等原生Kubernetes负载类型。
- 不支持DaemonSet：Serverless集群中不支持节点相关的功能。

## 弹性伸缩

ASK集群中没有真实节点，所以无需考虑节点的容量规划，也无需考虑基于cluster-autoscaler的节点扩容，您只需要关注应用的按需扩容。建议您配置HPA或者CronHPA策略进行Pod的灵活按需扩容。

## 网络管理

集群中的ECI Pod默认使用Host网络模式，占用交换机vSwitch的一个弹性网卡ENI资源，与VPC内的ECS、RDS互联互通。

- Service
支持创建LoadBalancer类型Service。
不支持NodePort类型Service：Serverless集群中不支持节点相关的功能。
- Ingress
SLB Ingress：无需部署Controller直接使用基于SLB七层转发提供的Ingress能力，请参见ingress示例。
Nginx Ingress：部署Nginx Ingress Controller后可以创建Nginx Ingress，请参见ingress-nginx示例。
- 服务发现
如果您的集群内部应用需要Service的服务发现功能，请在创建集群时开启Privatezone。

- 弹性公网IP
支持给ECI Pod挂载EIP，可自动创建或者绑定到已有的EIP实例。

## 存储管理

Pod支持挂载阿里云块存储和文件存储。

**阿里云块存储（Disk）**
使用flexvolume方式挂载：无需安装flexvolume插件。您可以选择指定diskId挂载，请参见disk-flexvolume-static.yaml示例；或者您也可以动态创建云盘，请参见disk-flexvolume-dynamic.yaml示例。
使用PV/PVC动态创建云盘后挂载：安装disk-controller后即可动态创建云盘后挂载，请参见disk-pvc-dynamic.yaml示例。

**阿里云文件存储（NAS）**
使用NFS volume：支持使用nfs方式挂载NAS目录，请参见nas-nfsvolume.yaml示例。
使用flexvolume静态挂载：无需安装flexvolume插件，直接指定NAS挂载地址，请参见nas-flexvolume.yaml示例。
使用PV/PVC静态挂载：安装disk-controller后即可使用PVC静态挂载NAS目录挂载，请参见nas-pvc.yaml示例。

## 日志管理

在ASK集群中无需部署logtail daemonset即可收集Pod的stdout和文件输出日志。

## 配置项及密钥管理

支持Secret和Configmap，以及通过volume挂载Secret和Configmap。

## 应用目录Chart管理

支持在应用目录部署Chart，连接Kubernetes生态应用。

