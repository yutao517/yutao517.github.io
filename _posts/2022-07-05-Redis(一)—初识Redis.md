---
layout: article
title: Redis(一)—初识Redis
tags: Redis
category: blog
date: 2022-07-05 18:17:00 +08:00
mermaid: true
---
## 简介

Redis是一种基于键值对(key-value)的NoSQL数据库

**SQL和NoSQL区别**

![image](https://user-images.githubusercontent.com/62100249/177453751-1f20409b-a133-46a1-8e8a-aa68b0f7ccb7.png)


**Redis特性**
- 基于键值对的数据结构服务器，value支持多种不同数据结构，功能丰富
- 单线程。每个命令具备一致性
- 低延迟，速度快（基于内存、IO多路复用、良好的编码）
- 支持数据持久化
- 支持主从集群，分片集群
- 支持多语言客户端

**Redis的典型应用场景**
- 缓存
- 排行榜系统
- 计数器应用
- 社交网络
- 消息队列系统
## 安装
**默认启动方式**
```bash
yum install gcc tcl -y
wget https://download.redis.io/releases/redis-6.2.7.tar.gz
mv redis-6.2.7.tar.gz /usr/local/src/
tar -zxvf redis-6.2.7.tar.gz
cd redis-6.2.7
make && make install
redis-server
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5b7c240168a84e4f89185bff0dd19c35.png)

**后台启动方式**

```bash
cp redis.conf redis.conf.default
vim redis.conf
```

```bash
#修改以下配置
bind 0.0.0.0
daemonize yes
#守护进程，修改为yes后台启动
requirepass 123456
#设置redis的访问密码
```

```bash
redis-server /usr/local/src/redis-6.2.7/redis.conf
ps -ef|grep redis
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b59dd9a62fb49149513418d1413d0f3.png)

**编写服务启动**

```bash
vim /etc/systemd/system/redis.service
```

```bash
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/src/redis-6.2.7/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
然后重载系统服务

```bash
systemctl daemon-reload
```
现在，我们可以用下面这组命令来操作redis了：

```bash
# 启动
systemctl start redis
# 停止
systemctl stop redis
# 重启
systemctl restart redis
# 查看状态
systemctl status redis
```
执行下面的命令，可以让redis开机自启：

```bash
systemctl enable redis
```

```bash
redis-cli -h 127.0.0.1 -p 6379 -a 123456
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b40d588361894049a2817297584af980.jpeg)

