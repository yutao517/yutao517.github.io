---
layout: article
title: Kubernetes(踩坑)—解决创建容器一直处于ContainerCreating状态
tags: Kubernetes
category: blog
date: 2022-04-28 21:00:00 +08:00
mermaid: true
---
对于这个问题，应该直接去查看详细信息，为什么无法启动。

```bash
kubectl get pods -o wide
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/166d5864639e4d25bcce502c4bf10d96.png)

发现node2上的容器无法启动，这时候我们在manager上选择node2上的一个容器去查看详细信息

```bash
kubectl describe pod k8s-nginx-6d779d947c-h6flq
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0041a81c3f784755a7f8d7725181d90a.png)

注意最下面这一行，意思是设置网桥失败: "cni0" 已经有一个不同于10.244.4.1/24的IP地址，这时候我们去node2上使用`ip add`去查看本机IP地址。

![在这里插入图片描述](https://img-blog.csdnimg.cn/b799b7609fe9402a89a74c26faf9c1ec.png)

这里我们注意到cni0的IP地址为10.244.2.1，当然不同，出现这种情况一般是重新加入集群，manager重新配置了网络插件。

在node2上去更改插件的IP地址
```bash
 vim /run/flannel/subnet.env 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/dce3ab15d9dc4b879fd883bf9621084d.png)

看到果然是10.244.4.1，改为10.244.2.1即可

回到manager，再次查看pod信息，效果很明显，已经启动。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7bae842b593f4a21958a91809aaad154.png)

当然原因也有其他的原因，根据容器创建详细信息来进行判断，希望能帮助到你。
