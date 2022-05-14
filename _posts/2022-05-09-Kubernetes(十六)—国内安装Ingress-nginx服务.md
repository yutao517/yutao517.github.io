---
layout: article
title: Kubernetes(十六)—国内安装Ingress-nginx服务
tags: Kubernetes
category: blog
date: 2022-05-09 21:40:00 +08:00
mermaid: true
---
## 背景

部署在 Kubernetes 集群中的应用暴露给外部的用户使用呢可以使用 NodePort 和LoadBlance 类型的Service把应用暴露给外部用户使用，除此之外，Kubernetes 还为我们提供了一个非常重要的资源对象可以用来暴露服务给外部用户，那就是 Ingress。对于小规模的应用我们使用 NodePort 或许能够满足我们的需求，但是当应用越来越多的时候，对于 NodePort 的管理就非常麻烦了，使用 Ingress 就非常方便了，可以避免管理大量的端口。

## 简介

Ingress 其实就是从 Kuberenets 集群外部访问集群的一个入口，将外部的请求转发到集群内不同的 Service 上，其实就相当于 nginx、haproxy 等负载均衡代理服务器，但是只使用nginx这种方式有很大缺陷，每次有新服务加入的时候需要改nginx 配置，不可能让我们去手动更改或者滚动更新前端的nginx-pod，那我们再加上一个服务发现的工具比如consul，Ingress 实际上就是这样实现的，只是服务发现的功能自己实现了，不需要使用第三方的服务了，然后再加上一个域名规则定义，路由信息的刷新依靠 Ingress Controller 来提供。

## 服务区别

service只能通过四层负载就是ip+端口的形式来暴露

- NodePort：会占用集群机器的很多端口，当集群服务变多的时候，这个缺点就越发明显
- LoadBalancer：每个Service都需要一个LB，比较麻烦和浪费资源，并且需要 k8s之外的负载均衡设备支持

ingress可以提供7层的负责对外暴露接口，而且可以调度不同的业务域，不同的url访问路径的业务流量。

- Ingress：K8s 中的一个资源对象，作用是定义请求如何转发到 service 的规则
- Ingress Controller：具体实现反向代理及负载均衡的程序，对Ingress定义的规则进行解析，根据配置的规则来实现请求转发，有很多种实现方式，如 Nginx、Contor、Haproxy等

## 工作原理
![在这里插入图片描述](https://img-blog.csdnimg.cn/32f8c0c83bc34d1f92145b634b14c152.png)

- 用户编写 Ingress Service规则， 说明每个域名对应 K8s集群中的哪个Service
- Ingress控制器会动态感知到 Ingress 服务规则的变化，然后生成一段对应的Nginx反向代理配置
- Ingress控制器会将生成的Nginx配置写入到一个运行中的Nginx服务中，并动态更新
- 然后客户端通过访问域名，实际上Nginx会将请求转发到具体的Pod中，到此就完成了整个请求的过程

## 安装
master节点
```bash
wget https://download.yutao.co/mirror/deploy.yaml
```
直接运行这个yaml文件会报错，因为默认从谷歌拉取镜像，我找了好多资料，都没有找到阿里云的镜像，所以只能采取曲线救国方式。
首先登录我的台北服务器去拉取所需镜像，再将镜像导出，最后再导入我的node节点

![在这里插入图片描述](https://img-blog.csdnimg.cn/28b0a8b9396148edb17daad00355533a.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0f19a03a28241d29910931ec890d6ea.png)

镜像已经放到下方链接，只需要将其下载下来导入到所有的node节点

```bash
wget https://download.yutao.co/mirror/controller
wget https://download.yutao.co/mirror/kube-webhook-certgen
```

```bash
docker load -i kube-webhook-certgen 
docker load -i controller
```
在master节点运行下载下来的yaml文件
```bash
kubectl apply -f deploy.yaml 
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8aaa86d5b8994faba9863156b2dd3e70.png)
