---
layout: article
title: 阿里云云原生容器工程师ACP认证(十九)—Knative Serverless应用
tags: 阿里云云原生容器工程师 Kubernetes Docker
category: blog
date: 2022-09-21 19:30:00 +08:00
mermaid: true
---

## Knative简介
**Knative** 是一款基于 Kubernetes 的 Serverless 框架，其目标是制定云原生、跨平台的Serverless编排标准。

角色
- 基于 Kubernetes的 Serverless 解决方案
- 标准化 Serverless技术架构
- 简化学习成本

组件
- Build：源到容器的构建和编排
- Event：消息传递层，事件交付管理
- Serving：请求计算，基于负载自动伸缩

优势
- 便利性
- 生态成熟
- 标准化
- 自动伸缩
- 服务间解耦
- 应用监控

## ASK Knative优势

![在这里插入图片描述](https://img-blog.csdnimg.cn/4b48e4ebdf3b40fdbbd2c9475b878850.png)

## ASK Knative 最佳实践

观测服务的QPS、RT和Pod扩缩容趋势
- 配置Logstore
- 配置QPS统计
- 配置RT统计
- 配置Pod扩缩容趋势统计
- 观测服务运行状况

观测服务的CPU和Memory使用情况
- ASK集群已开通Knative功能
- 集群已开通阿里云Prometheus监控功能
- 配置Prometheus监控目标ASK集群

## 方案背景

![在这里插入图片描述](https://img-blog.csdnimg.cn/123644e3e61b408d833173ceaa8d7f43.png)


**场景需求**

- 基于Jenkins构建自动化CI／CD集群系统
- 集群资源合理利用，控制成本
- 集群高可用性需求
- 集群资源快速弹性伸缩

**方案优势**

- 高可用服务
- 自动弹性伸缩，资源合理利用
- 可扩展性好

**解决问题**

- 集群Master节点单点故障
- 集群资源利用率低
- 集群资源可扩展性差

## 方案架构

**服务高可用**

避免Master单点故障导致集群流程不可用

**弹性伸缩**

每次运行Job时，自动构建Jenkins Slave，Job完成以后，自动注销并删除容器，资源自动释放，节省成本。

**资源合理利用**

- 动态分配Slave到空闲节点
- 降低出现由于节点资源限制的排队等待情况

**扩展性好**

- 集群资源严重不足时，可以快速添加节点
- 降低集群资源不足导致Job排队等待情况

![在这里插入图片描述](https://img-blog.csdnimg.cn/cf4e7608c8ce473088bdcc4350ded6e2.png)
