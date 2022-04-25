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

**浏览器访问**

![在这里插入图片描述](https://img-blog.csdnimg.cn/31a4dcfc06cc43cb9a8a3584569376f5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20438fcd94a648a59708f0bfbf7d4ffa.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/5ae22a8fe6a8461d9e1b482153113ac2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/82ae8ebf8522438d9a76a592e4e7d829.png)
实际上，改了一台manager的web内容，访问manager是随机性页面，除非三台主机都修改web页面。所以内部可能有负载均衡机制。


## Swarm集群使用NFS
![在这里插入图片描述](https://img-blog.csdnimg.cn/99d1cb1b3a2f4fa893306b4dd135391f.png)

**容器使用NFS**

nfs服务器（192.168.2.58）
```bash
yum install nfs-utils -y
service nfs-server start
mkdir /web
vim /web/index.html
>wang
exportfs -rv
chmod a+w /web
```
容器manager主机

```bash
docker service create --name nfs-service     --mount 'type=volume,source=nfsvolume,target=/usr/share/nginx/html,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/web,"volume-opt=o=addr=192.168.2.58,rw,nfsvers=4,async"' -dp 8010:80 --replicas 3 nginx:latest
```
可以看到使用nfs服务器页面一致

![在这里插入图片描述](https://img-blog.csdnimg.cn/47cee75acfbd44d79f454a8bac884e04.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/50e6cd4114464f05bb1eb32a688f510e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f1214f24dd2248d39a146f39514c796f.png)

## 退群
worker主机
```bash
docker swarm leave 
#退出swarm集群
```
manager主机

```bash
docker node ls
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/be20cdf41749414cb695e946ef46b6c4.png)

删除down掉的主机

```bash
docker node rm azure
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5f423e587afa4dfb8d62aa371f393de8.png)

## 滚动升级

```shell
docker service update --image nginx:latest nginx
#升级nginx服务的镜像到最新状态
```

