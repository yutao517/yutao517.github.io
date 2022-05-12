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
