---
layout: article
title: Docker(十二)—Swarm集群共享数据
tags: Docker
category: blog
date: 2022-04-23 21:40:00 +08:00
mermaid: true
---
## 环境
保证三台主机已经在一个swarm集群，还有一台nfs服务器取数据。

## Swarm集群使用Volume
**manager创建一个新的volume**

```bash
docker volume create yutao
#创建数据卷yutao
```
**编辑数据卷内容**
```bash
vim /var/lib/docker/volumes/yutao/_data/index.html
>yutao
```
**查看volume**

```bash
 docker volume ls
 docker volume inspect yutao
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a9fae9c2f13a4647a6155616b7b05d57.png)

**启动nginx服务挂载数据卷**
```bash
docker service create -d --replicas 3 --mount type=volume,src=yutao,dst=/usr/share/nginx/html --name  wyt-nginx-swarm -p 8010:80 nginx
```
 **查看服务**

```bash
 docker service ls
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/c7b347c8e29348d69c0b19ff7b11605f.png)

**浏览器访问都成功挂载数据卷**

![在这里插入图片描述](https://img-blog.csdnimg.cn/31a4dcfc06cc43cb9a8a3584569376f5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5ae22a8fe6a8461d9e1b482153113ac2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/82ae8ebf8522438d9a76a592e4e7d829.png)


## Swarm集群使用NFS
[搭建NFS服务器过程](https://blog.yutao.co/blog/2022/04/12/Nginx-%E6%90%AD%E5%BB%BANFS%E6%9C%8D%E5%8A%A1%E5%99%A8.html)
![在这里插入图片描述](https://img-blog.csdnimg.cn/99d1cb1b3a2f4fa893306b4dd135391f.png)

**容器使用NFS**

nfs服务器
```bash
yum install nfs-utils -y
service nfs-server start
vim /etc/exports
>/web 192.168.2.0/24(rw,all_squash,sync)
mkdir /web
vim /web/index.html
>yutao 
exportfs -rv
chmod a+w /web
```
容器manager主机

```bash
docker volume create nfs
#创建数据卷
mount 192.168.2.58:/web /var/lib/docker/volumes/nfs/_data/
#数据卷挂载到NFS服务器
docker service create -d --replicas 3 --mount type=volume,src=nfs,dst=/usr/share/nginx/html --name  nfs-nginx -p 8010:80 nginx
#启动服务
```
