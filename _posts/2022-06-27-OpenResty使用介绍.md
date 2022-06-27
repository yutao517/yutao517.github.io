---
layout: article
title: OpenResty使用介绍
tags: Nginx OpenResty
category: blog
date: 2022-06-27 17:48:20 +08:00
mermaid: true
---
## 简介

OpenResty是一个基于Nginx的Web平台，可以使用其LuaJIT引擎运行Lua脚本。该软件由章亦春创建。2011年之前，它最初由淘宝网赞助，2012年至2016年主要由Cloudflare支持。自2017年起，主要得到OpenResty软件基金会和OpenResty公司的支持。

OpenResty旨在构建可扩展的Web应用、Web服务和动态Web网关。OpenResty的架构是基于几个nginx模块，这些模块已经被扩展，以便将nginx扩展为一个web应用服务器，处理大量的请求。OpenResty解决方案的概念旨在完全在nginx服务器中运行服务器端的Web应用，利用nginx的事件模型，不仅与HTTP客户端进行非阻塞的I/O，而且与MySQL、PostgreSQL、Memcached和Redis等远程后端进行非阻塞I/O。

## OpenResty部署

```bash
yum install readline-devel pcre pcre-devel openssl openssl-devel gcc curl GeoIP-devel
wget https://openresty.org/download/openresty-1.19.3.1.tar.gz
tar -zxvf openresty-1.19.3.1.tar.gz
cd openresty-1.19.3.1
./configure 
make && make install
```

```bash
./nginx/sbin/nginx
ps -ef|grep nginx
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2878d5b0ee7940b19d5703033ce2ed42.png)

## 官方示例Hello World
[官方文档:http://openresty.org/cn/getting-started.html](http://openresty.org/cn/getting-started.html)

```bash
cd /usr/local/openresty
cp nginx/conf/nginx.conf nginx/conf/nginx-backup.conf
vim nginx/conf/nginx.conf
```

```bash
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua_block {
                ngx.say("<p>hello, world</p>")
            }
        }
    }
}
```
**启动 nginx**

```bash
./nginx/sbin/nginx
ps -ef|grep nginx
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4a240a792da44878918b04ed2d4de260.png)
## 浏览器访问验证
因为我使用的是virtualbox虚拟机，所以要对其进行网络端口转发。

![在这里插入图片描述](https://img-blog.csdnimg.cn/10f3d868451244fab1a7cd50a10602bc.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/000148b1b57f456e8348688055466314.png)
