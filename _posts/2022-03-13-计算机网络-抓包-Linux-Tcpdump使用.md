---
layout: article
title: 计算机网络-抓包-Tcpdump
tags: 计算机网络
category: blog
date: 2022-03-13 00:00:00 +08:00
mermaid: true
---

## 概念
**tcpdump**是一个运行在命令行下的数据包分析器。它允许用户拦截和显示发送或收到过网络连接到该计算机的TCP/IP和其他数据包(Unix操作系统的抓包工具)

## 常用命令

监视指定网卡ens33的数据	

```bash
tcpdump -i ens33
```

 监视指定主机192.168.2.178的数据包
```bash
tcpdump -i ens33  host 192.168.2.178
```

监视192.168.2.178和192.168.2.14或192.168.2.15之间通信的数据包
```bash
tcpdump -i ens33 host 192.168.2.178 and \(192.168.2.14 or 192.168.2.15\)
```
监视192.168.2.178与任何其他主机之间通信的IP数据包,但不包括与192.168.2.14之间的数据包
```bash
tcpdump -i ens33 host 192.168.2.178 and not 192.168.2.14
```
监视主机192.168.2.14 发送的所有数据
```bash
tcpdump -i ens33 src host 192.168.2.14
```
监视所有发送到主机192.168.2.14的数据包

```bash
tcpdump -i ens33 dst host 192.168.2.14
```
监视主机192.168.2.178和端口80的数据包

```bash
tcpdump -i ens33 port 80 and host 192.168.2.178
```

抓取到本机22端口包

```bash
tcpdump  -i ens33 tcp and dst port 22 -nn
```
抓取192.168.2.14的ping包
```bash
tcpdump -i ens33 icmp and src 192.168.2.14 -nn 
```

监视192.168.2.14访问主机80端口的数据
```bash
tcpdump -i ens33 src host 192.168.2.14 and dst port 80 -nn
```

