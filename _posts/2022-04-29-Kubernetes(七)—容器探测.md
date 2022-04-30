---
layout: article
title: Kubernetes(七)—容器探测
tags: Kubernetes
category: blog
date: 2022-04-29 22:08:00 +08:00
mermaid: true
---
[参考文档](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
## 容器类型

k8s已经支持四种类型的container：
- 标准容器
- sidecar容器：代理，适配器，增强主容器功能
- init容器：应用容器运行前必须先运行完成的一个或多个初始化容器，init容器与普通的容器非常像，除了如下两点：它们总是运行到完成。每个都必须在下一个启动之前成功完成。
- Ephemeral容器：临时容器，临时容器与其他容器的不同之处在于，它们缺少对资源或执行的保证，并且永远不会自动重启，因此不适用于构建应用程序

## 容器探测
容器探测是Pod在运行时，由Kubelet对容器进行周期性的健康诊断。

**检查机制** 
- exec
在容器内执行指定命令。如果命令退出时返回码为 0 则认为诊断成功。
- grpc
使用 gRPC 执行一个远程过程调用。 目标应该实现 gRPC健康检查。 如果响应的状态是 "SERVING"，则认为诊断成功。 gRPC 探针是一个 alpha 特性，只有在你启用了 "GRPCContainerProbe" 特性门控时才能使用。
- httpGet
对容器的 IP 地址上指定端口和路径执行 HTTP GET 请求。如果响应的状态码大于等于 200 且小于 400，则诊断被认为是成功的。 
- tcpSocket
对容器的 IP 地址上的指定端口执行 TCP 检查。如果端口打开，则诊断被认为是成功的。 如果远程系统（容器）在打开连接后立即将其关闭，这算作是健康的。

**探测类型** 
按照容器探测的类型进行分类，容器探测可以分为以下两种：
- **存活性探测（livenessProbe）**
存活性探测用于判定Pod是否处于“健康”状态，如果判定Pod中的容器处于不健康状态，kubectl将会杀死容器并根据重启策略来决定是否对该容器进行重启。未定义存活性检测的容器的默认状态为Success。
- **就绪性探测（livenessProbe）**
就绪性探测用于判断容器是否准备就绪，是否可以为外部提供服务。如果就绪性探测失败，Pod的端点控制器（如Service）就会将该Pod的IP移除，从而使得外部流量避免访问该未就绪的容器，当就绪性检测成功后，其IP会被添加回来。如果容器不提供就绪态探测，则默认状态为 Success。
- **启动性探测（startupProbe）**
启动性探测用于判断容器是否启动。如果启动探测失败，kubelet 将杀死容器，而容器依其重启策略进行重启，如果容器没有提供启动探测，则默认状态为 Success

**探测结果** 
每次探测都将获得以下三种结果之一
- Success（成功）
容器通过了诊断。
- Failure（失败）
容器未通过诊断。
- Unknown（未知）
诊断失败，因此不会采取任何行动。


