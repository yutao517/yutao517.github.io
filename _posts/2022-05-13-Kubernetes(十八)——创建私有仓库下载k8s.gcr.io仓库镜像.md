---
layout: article
title: -Kubernetes(十八)——创建私有仓库下载k8s.gcr.io仓库镜像
tags: Kubernetes
category: blog
date: 2022-05-13 20:39:00 +08:00
mermaid: true
---
今天在按照[官方文档](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)部署ingress-nginx服务的时候，需要部署hello,world应用程序

![在这里插入图片描述](https://img-blog.csdnimg.cn/42be78ce48174ca78448c1208f269ea8.png)

但是官方文档提供的镜像网址是无法在国内打开的，而且并没有提供yaml文件，这时候如果你有一台海外服务器，可以把镜像拉取下来，导出再导入另一台服务器，但是如果镜像过大，需要很长时间，这时候就需要想其他办法，而且并不是每个人都有一台国外服务器，所以使用阿里云的容器服务是不错的选择。

## 创建代码仓库
如果有条件仓库可以选择github，如果github无法访问建议使用[阿里云仓库](https://code.aliyun.com/)

**代码库绑定阿里云账号**
[https://help.aliyun.com/document_detail/202197.html](https://help.aliyun.com/document_detail/202197.html)

**创建代码库**

在仓库master分支新建/mirror/Dockerfile文件
```bash
FROM gcr.io/google-samples/hello-app:1.0
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/40a461f367d94f4a94e58ce302cfb79d.png)

## 创建镜像服务


[阿里云容器镜像服务](https://cr.console.aliyun.com/cn-hangzhou/instances)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c92c86365d454d2a99c4d02bd5e339a7.png)

如图所示，创建个人实例，然后进入实例创建命名空间。

![在这里插入图片描述](https://img-blog.csdnimg.cn/46f055660c2643a6aaade6ad2c89a0e0.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/1535aab9c784491faf16131251432f33.png)

创建成功后进入镜像仓库，创建镜像仓库

![在这里插入图片描述](https://img-blog.csdnimg.cn/98ba58651bcb4ff689be2606eb48caef.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/01d44cfc48a142868d4d10005034e907.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ace69ac8c38c4f609be2824aa49bc79d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ea24d490ee6443718859f2e0ff8be1d5.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0b6d2ede80b2403bb8ae65ecb8a69640.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/dec4ac1991f24c39aa9db30da1a7d6b3.png)

## 拉取镜像

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/yutao517/container:1.0
docker tag registry.cn-hangzhou.aliyuncs.com/yutao517/container:1.0  gcr.io/google-samples/hello-app:1.0
```

这里接上面的镜像版本，可以看到该镜像已下载

![在这里插入图片描述](https://img-blog.csdnimg.cn/fbd2de6375a34faf9ede3ba8fe58bc51.png)

然后换标签，可以看到更新成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/4b8f4311492c43aa81069c37aed2b81f.png)
