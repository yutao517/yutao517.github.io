---
layout: article
title: Kubernetes(三)—Pod对象部署和应用
tags: Kubernetes
category: blog
date: 2022-04-27 14:14:00 +08:00
mermaid: true
---
## Pod资源创建

```bash
kubectl create deployment k8s-nginx --image=nginx -r 8
#以nginx为镜像创建名为k8s-nginx的8个副本的pod资源
```

## Pod资源查看

```bash
kubectl get deployment
```
```bash
kubectl get pod
```

>接 -o wide参数，可以查看该Pod对象的详细信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/d52e164ffd0147948c54b1cc5f0da9ae.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/1a3802763fba42988ef35ec95b009fa4.png)

## Pod资源访问

master访问10.244.1.4

![在这里插入图片描述](https://img-blog.csdnimg.cn/9b78f2c83a6d498a82c830fb3ca7268c.png)

nginx服务已经在k8s集群中启动，但是并没有暴露端口，所以外界无法访问。

## Pod扩缩

```bash
kubectl scale deployment/k8s-nginx --replicas 10
#扩容到10个pod也可以缩小
```
## Pod节点删除

```bash
kubectl delete pod k8s-nginx-6d779d947c-5zt6x 
#删除名为k8s-nginx-6d779d947c-5zt6x的pod
```
在Kubernetes集群中，由于该Pod被Controller控制器所控制，因此我们尽管能够删除该Pod对象，但是replica controller副本控制器会再次创建Pod对象。如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/3e228acc4b884372a38409493fb2dcbd.png)
