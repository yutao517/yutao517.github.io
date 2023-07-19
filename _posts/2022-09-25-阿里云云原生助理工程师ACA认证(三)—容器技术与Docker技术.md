---
layout: article
title: 阿里云云原生助理工程师ACA认证(三)—容器技术与Docker技术
tags: 阿里云云原生容器工程师 Kubernetes Docker
category: blog
date: 2022-09-25 14:31:00 +08:00
mermaid: true
---
## 容器技术介绍与特点
**集装箱**

集装箱的出现，改变了全球化运输模式，托运货物的人只需要保证货物在集装箱内的密封和固定，而无需关心集装箱如何被摆放和运输。
- 标准统一
- 封闭隔离
- 运输简化
- 降低成本

**集装箱对容器技术的核心启示**

容器的核心思想:

- 将集装箱的思想应用到了软件的打包和部署上，为各类不同的代码提供了一个基于容器的标准化运输系统。
- 容器可以将任何应用及其依赖环境打包为一个轻量级、可移植、自包含的独立运行环境
- 容器可以运行在几乎所有的操作系统之上

**IT世界的容器技术特点**

- 容器是自包含的
- 容器是可移植的
- 容器是互相隔离的
- 容器是轻量级的

## 容器和虚拟机之间的差异

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/466b657a-9677-4504-a27e-07a97c6a43e3)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/d43704b2-7378-4e8f-9c2d-67d46577a290)


## Docker的核心概念及架构
**Docker容器**

- 是资源分割和调度的基本单位
- 封装整个服务的运行时环境，用于构建、发布和运行分布式应用的一个框架
- 是一个跨平台、可移植并且简单易用的容器解决方案
- Docker是世界领先的软件容器平台。
- Docker使用Google公司推出的Go语言进行开发实现，基于Linux内核的**cgroup，namespace**，对进程进行封装隔离。
- Docker能够自动执行重复性任务，例如搭建和配置开发环境，从而解放了开发人员以便他们专注在构建杰出的软件。
- 用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

**为什么docker成了容器的事实标准?**

- 一致的运行环境(开发在自己PC跑没问题，在测试环境有问题)
- 秒级启动，更快速的启动时间
- 资源隔离，容器之间相互隔离不受影响
- 弹性伸缩、快速扩展
- 迁移方便，无需担心运营环境
- 持续交付和部署，提升交付效率

**Docker容器到底帮助我们解决什么问题?**

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/f3f6f317-c3bc-4615-96b3-50005008d8a2)

- 简化配置
- 代码流水线管理
- 提高开发效率
- 隔离应用
- 整合服务器
- 调试能力
- 多租户环境
- 快速部署

**Docker的三大核心概念**

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/4545fcde-50ec-45cf-806e-13092124767c)

**容器镜像-容器封装的标准基础**

 **Docker镜像**：是一个Linux的文件系统，包含运行的程序以及程序、库、资源、配置等数据。
 
 Docker镜像
 
 - 类似于一个只读压缩包
 - 用于创建容器
 - 分层构建，前一层时后一层的基础
- 包含了一个精简的操作系统、应用运行所必须的文件和依赖包
- 一次构建，到处运行

镜像构建(Build)的方法:
- docker commit生成镜像
- Dockerfile(建议)

 **容器-镜像创建运行实例进程**

**Docker中的容器和镜像有什么区别?**
- 容器是镜像的运行实体
- 容器可以被创建、启动、停止、删除、暂停等
- 容器运行特定应用
- 容器之间相互隔离
- 容器层是多层只读镜像之上运行的可写层
- 容器停止/删除，可写层数据丢失
- 数据持久化：使用宿主存储/网络存储

**镜像仓库-镜像文件存储的场所**

- Docker镜像仓库类似于代码仓库，它是Docker集中存放镜像文件的场所。
- 实现Docker镜像的全局存储提供API接口
- 提供Docker镜像的下载/推送/查询

> docker pull 镜像下载 
> docker push镜像推送 
> docker search镜像查询

## 容器Docker体系结构详解
**Docker Registry：镜像仓库**

- 官方的仓库：Docker Hub
- 私有镜像仓库搭建：基于开源项目Harbor或阿里云提供的镜像仓库服务

**Docker Server：Docker服务端**

- 宿主机中运行守护进程(Docker daemon)
- 接收Docker客户端(Client)发送的指令来执行拉取
- 镜像、缓存、启动容器等操作

**Docker Client: Docker客户端**

- 主要负责通过docker命令行对容器进行基本操作，如拉取镜像，构建镜像，运行容器等等

**Build,Ship, and Run Any App,Anywhere(随时随地的构建发布运行任何APP)**

Docker运行过程也就是去仓库把镜像拉到本地，然后用一条命令把镜像运行起来变成容器。我们也常常将Docker称为码头工人或码头装卸工，这和Docker的中文翻译搬运工人如出一辙。

- Build(构建镜像)︰镜像就像是集装箱，包含文件以及运行环境等等资源;
- ship(运输镜像)︰在宿主机和仓库间进行运输，这里仓库就像是超级码头;
- Run(运行镜像):运行的镜像就是一个容器,容器就是运行程序的地方。

## 什么是Dockerfile

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/80554c05-59d8-408c-adaa-0f93ab84e2ea)

- Dockerfile是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。
- Dockerfile是软件开发的基础Docker镜像是软件的交付品
- Docker容器则可以认为是软件的运行态
- Dockerfle面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，构成Dockerd的基础

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/623fb669-318a-4c8e-b428-27b17378e44f)

## Docker基础操作

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/f6767a7d-cc7d-4a80-84ae-2d0e56f500d9)


