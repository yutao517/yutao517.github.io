---
layout: article
title: Docker(十一)—Swarm 集群管理
tags: Docker
category: blog
date: 2022-04-22 10:40:00 +08:00
mermaid: true
---
## 简介

Docker Swarm是Docker公司推出的的集群管理工具，几乎全部用GO语言来完成的开发
Docker Swarm 和 Docker Compose 一样，都是 Docker 官方容器编排项目，但不同的是，Docker Compose 是一个在单个服务器或主机上创建多个容器的工具，而 Docker Swarm 则可以在多个服务器或主机上创建容器集群服务，对于微服务的部署，显然 Docker Swarm 会更加适合。

**工作原理**
swarm 集群由管理节点(manager)和工作节点(work node)构成。
swarm mananger：负责整个集群的管理工作包括集群配置、服务管理等所有跟集群有关的工作。
work node：即图中的 available node，主要负责运行相应的服务来执行任务(task)。

![在这里插入图片描述](https://img-blog.csdnimg.cn/ce32f939002b408398306683a6140413.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**Swarm的调度策略**
- Random：随机选择一个Node来运行容器
- Spread：选择运行容器最少的那台节点来运行新的容器
- Binpack：选择运行容器最集中的那台机器来运行新的节点

## 实验
准备三台主机，都安装好docker，关闭防火墙，开放2377/tcp（管理端口）、7946/udp（节点间通信端口）、4789/udp（overlay 网络端口）端口。
manager：192.168.2.248
worker-1：192.168.2.249
worker-2：192.168.2.250

**manager主机初始化管理节点**
```bash
docker swarm init --advertise-addr 192.168.2.248
#将192.168.2.248初始化为管理节点
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/546c7838509742639e96f5d5c7a912e1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**在worker-1 ,worker-2输入提示命令加入swarm集群**
```bash
docker swarm join --token SWMTKN-1-4ub540qnlteq52z27tpzenupj539dgzmysn88hcmmynqux72lr-dlgvvd0ly92xpszv3tzv0u72e 192.168.2.248:2377
```

```bash
docker node ls
#查看集群的主机
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/cdf5fe8670754428aec81aadd6a08367.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**manager主机创建服务**

```bash
docker images
#查看自己的镜像
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0e641102ffad460b9de541d8fea1d118.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

我使用昨天创建的WordPress镜像，启动服务

```bash
docker service create --replicas 10 --name wordpress -p 7000:80 wordpress

#使用wordpress为镜像，名为wordpress的服务，启动10个实例，将宿主机的7000端口映射到容器的80端口
```

```bash
docker service ls
#查看运行的服务
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ad404ac5c6c5401ea395faf72ef3078c.png)
**查看运行的容器实例**

```bash
docker ps
#在各个主机分别运行该命令查看运行的容器实例
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/28c98770e9534b078c1ce3628d66b1b3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c88fd55b9fac4d07811cfbccf5d02b78.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/441f702712b24b2ea806aba7987ad315.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**浏览器访问页面**

![在这里插入图片描述](https://img-blog.csdnimg.cn/fa5917e1b8454e0c971725f8404c3ca8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9b4197b25d294edda75075eb0f8fe4fd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a66637ba32f4130b9418666c7ed14dc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**增加容器实例**
```bash
docker service update --replicas 15 wordpress
#实例增加到15个
docker service ls
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5b4d843cf8164b9d9f23625a55ed51f2.png)

查看每台主机运行的容器，可以看到每台主机有5个容器实例。当一台主机宕机，容器实例会转移到另外两台，实现初步的高可用，但是当主机状态恢复并没有重新分配。

**删除服务**
```bash
docker service rm wordpress
```
结束实验
