---
layout: article
title: Prometheus—grafana使用
tags: Prometheus
category: blog
date: 2022-04-25 11:58:00 +08:00
mermaid: true
---
## 简介

Grafana是一个跨平台、开源的数据可视化网络应用程序平台。用户配置连接的数据源之后，Grafana可以在网络浏览器里显示数据图表和警告。

## 安装

```bash
vim /etc/yum.repos.d/grafana.repo
```

```bash
[grafana]
name=grafana
baseurl=https://packages.grafana.com/enterprise/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

```bash
yum install grafana-enterprise -y
```

```bash
systemctl start grafana-server
systemctl enable grafana-server
#开机自启动
```
```bash
http://192.168.2.248:3000/
浏览器访问服务器IP的3000端口
```

> 默认的用户名:admin 密码:admin

## 配置

**添加监控主机**

设置--->Data sources--->Add data sources
选择Prometheus

![在这里插入图片描述](https://img-blog.csdnimg.cn/b7f2e12c03ec4bdaa82bfe561303ed6f.png)

输入监控主机IP端口保存

**添加监控项**

![在这里插入图片描述](https://img-blog.csdnimg.cn/40f04b65b6d840feaf4e879a7c68530c.png)

在这里add a new panel可以选择一些数据例如CPU、memory。

![在这里插入图片描述](https://img-blog.csdnimg.cn/8054ed4a241f485da280aa668d098919.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3d6484f746b84bf08f20b07973d7411d.png)



## 使用模版

[官网模版](https://grafana.com/grafana/dashboards) 

这里我使用了这个模版，download json文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/706ed7e763c94d97b4b9fb9bf42eb186.png)

下载好导入json文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/a11ec7efe328478cb4c99b83331da1e8.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6b4ff14294144515b16b855afc3d96c7.png)

