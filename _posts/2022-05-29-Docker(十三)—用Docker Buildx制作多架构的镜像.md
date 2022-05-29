---
layout: article
title: Docker(十三)—用Docker Buildx制作多架构的镜像 
tags: Docker
category: blog
date: 2022-05-59 13:22:00 +08:00
mermaid: true
---
## 背景

默认情况下，x86_64平台只能构建x86_64镜像，如果需要在X86_64平台构建多平台镜像（比如ARM64），我们可以用Docker官方提供的Buildx工具来完成多平台镜像构建。ARM 架构与X86相比，ARM 低功耗、移动市场占比高，X86 高性能、服务器市场占比高。
构建时要用到 docker buildx 命令，Docker 版本需要 19.03+

## 初始化Docker Buildx
Docker Buildx属于实验性功能，默认并没有开启，需要修改

```bash
vim /etc/docker/daemon.json
```
添加一行
```bash
"experimental": true
```
由于Docker默认的builder实例不支持同时指定多个--platform，所以必须先创建一个新的builder实例，并使用。
```bash
docker buildx create --name builderx
docker buildx use builderx
docker buildx inspect --bootstrap
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/71d575fe81194e24b58fb7fc9b939232.png)

Docker 在 Linux/AMD64 系统架构下是不支持 ARM 架构镜像，因此我们可以运行一个新的容器（Emulator）让其支持该特性。

```bash
docker run --rm --privileged tonistiigi/binfmt:latest --install all
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/16802f60963549b4a8e6b88214c538e9.png)
## 制作镜像
制作一个简单的Nginx多架构镜像

```bash
vim Dockerfile
```

```bash
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2d9ebd4bff744d4e87e3bdb60ca191da.png)

镜像构建后默认保存在构建缓存中，没有保存在本地，所以将 type 指定为 docker，必须分别为不同的 CPU 架构构建不同的镜像，不能合并成一个镜像。

```bash
docker buildx build -t harbor.yutao.co/library/nginx-arm64 --platform=linux/arm64 -o type=docker .
docker buildx build -t harbor.yutao.co/library/nginx-amd64 --platform=linux/amd64 -o type=docker .
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a74801e882b24c28b9847d123fce5fa4.png)

看到我们制作的镜像

## 检验镜像架构

#### 上传到Harbor仓库

上传镜像到私有仓库进行检验

```bash
docker login harbor.yutao.co
#填写自己的私有仓库地址
docker push harbor.yutao.co/library/nginx-amd64 
docker push harbor.yutao.co/library/nginx-arm64   
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed478e4fed564dc18340bd54076bd6c4.png)

上传成功后查看属性看到架构为amd64

![在这里插入图片描述](https://img-blog.csdnimg.cn/e540ca8760ba40fe8d1f32e7a008174d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/cf0e13df21184575b9131483e56953b1.png)

可以看到架构的不同

#### 上传到Docker hub仓库
在Docker Hub创建library仓库

![在这里插入图片描述](https://img-blog.csdnimg.cn/cd6a1963f16f45049ccaf8755cff3258.png)

```bash
docker login
```

```bash
docker buildx build --platform linux/arm64,linux/amd64 -t yutao517/library . --push
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/86bcd284a3d14a5a80d50c1d3a4579d0.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/005639b577294775be26e3952c39da08.png)

#### 上传到阿里云镜像仓库
创建library本地仓库

![在这里插入图片描述](https://img-blog.csdnimg.cn/5389cbc4c1bc4e1b866e9320a1126c61.png)

```bash
docker login --username=tao30564 registry.cn-hangzhou.aliyuncs.com
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a32649f7cfe34ea9a372d4de03e39187.png)
