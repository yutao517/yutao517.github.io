---
layout: article
title: Nginx-dns-域名-解析过程-搭建自己的DNS服务器
tags: Nginx 计算机网络
category: blog
date: 2022-03-25 000000 +0800
mermaid: true
---

## 域名系统

**域名系统**（英语：Domain Name System，缩写：DNS）因特网使用的命名系统，能够使人更方便地访问互联网，用来把人便于记忆的机器名字转换为IP地址。DNS服务是和HTTP协议一样属于**应用层**，DNS在进行区域传输的时候使用**TCP**协议，其它时候则使用**UDP**协议。

## 域名


![在这里插入图片描述](https://img-blog.csdnimg.cn/9bcbdf8ed8f24d52ac675cd09c09cc9b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)


 |  sc   | .yutao  |.co
 |  ----  | ----  | ---|
 |三级域名| 二级域名| 顶级域名|

 **类型**
 
 域名分为顶级域名和二级域名，顶级域名又可分为国际域名和国内域名，二级域名分为类别域名和行政区域名。
 
 国际顶级域名
 |  后缀   | 类型|
   |  ----  | ----  | 
 |.org|非盈利组织 |
  |.edu|教育机构 |
   |.com|工商企业 |
   |.net|网络提供商|
   |.gov|政府机构
   |.mil|美国军事部门域名
   
   国内顶级域名
   
   |  后缀   | 类型|
   |  ----  | ----  | 
 |.cn|中国|
  |.hk|中国香港 |
   |.tw|中国台湾 |
   |.us|美国|
   |.jp|日本|
   |.kr|韩国|
   


## DNS服务器
**作用**：
- 正向解析---根据域名查找IP地址
- 反向解析---根据IP地址查找域名

**级别**
- 本地域名服务器：自己的主机填的DNS服务器地址，比如114.114.114.114
- 根域名服务器：最高级别的域名服务器，负责返回顶级域的权限域名服务器地址。全球一共13台，10台在美国，1台日本，一台瑞典，一台英国，设镜像服务器可解决该问题，并使得实际运行的根域名服务器数量大大增加。
- 顶级域名服务器：国际顶尖域名服务器，如.com .cn .org
- 权限域名服务器：负责维护一个区域的域名信息，如yutao.co

**类型**
- 主域名服务器特定DNS区域的官方服务器，具有唯一性，权威性，负责维护一个区域所有域名信息。
- 缓存域名服务器：又叫唯高速缓存服务器，当从远程域名服务器获得域名解析信息后，将其缓存到本地中，当下次需要请求相同的域名解析时，直接从本地缓存中读取，缓存域名信息不具有权威性，提高重复查询的速度。
-  辅助域名服务器：又叫从域名服务器，当主域名服务器出现故障，关机或负载过重等情况，辅助域名服务器作为备份服务器来提供域名解析服务，辅助域名服务器是从主域名服务器下载的所有域名信息，域名信息不具有修改权限。
- 转发域名服务器
负责所有非本地域名的本地查询。转发域名服务器接到查询请求后，在其缓存中查找，如找不到就将请求依次转发到指定的域名服务器，直到查找到结果为止，否则返回无法映射的结果

## DNS解析过程![在这里插入图片描述](https://img-blog.csdnimg.cn/8d9206cb845c40b39e9b9ef27e6d8d99.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
输入www.yutao.co时：


 - 1.浏览器先检查自身缓存中有没有被解析过的这个域名对应的ip地址，如果有，解析结束。

 - 2.如果浏览器缓存中没有，浏览器会检查操作系统缓存，就是/etc/host文件，如果有，解析结束。

 - 3.如果host文件中没有，主机向其本地DNS服务器进行递归查询，一般这台服务器距离你不会很远，而且缓存了大量域名解析结果，大约80%的域名解析到这里就完成了。

 - 4.如果本地域名服务器仍然没有，采用迭代的查询，向根域名服务器查询。

 - 5.根域名服务器告诉本地域名服务器，下一次应该查询的顶级域名服务器的地址,比如,去找.co。

 - 6.本地域名服务器向顶级域名服务器.co发起查询。

 - 7.顶级域名服务器告诉本地域名服务器，下一次应查询的权限域名服务器地址yutao.co

 - 8.本地域名服务器向权限域名服务器yutao.co发起查询。

 - 9.权限域名服务器根据映射关系表，告诉本地域名服务器所查询的yutao.co服务器的IP地址

 - 10.本地域名服务器缓存了yutao.co域名和对应的IP地址。

 - 11.本地域名服务器把解析的结果返回给用户。


**迭代查询和递归查询**
 - 递归查询：如果主机所询问的本地域名服务器不知道被查询的域名的IP地址，本地域名服务器向根域名服务器继续发出查询请求报文帮查，而不是让主机自己进行下一步查询，只有 IP地址和无法查询两种报错。
  - 迭代查询：根域名服务器通常是把自己知道的顶级域名服务器的IP地址告诉本地域名服务器，让本地域名服务器自己向顶级域名服务器查询。 
  
## DNS服务器搭建

1.缓存域名服务器
  
服务机配置
```bash
 yum install bind -y #下载bind,bind是linux系统下的一个DNS服务程序
 service named start #启动
 ps aux|grep named #查看进程
 ss -anplut|grep named #查看端口号
 yum install bind-utils -y #bind-utils是bind软件提供的一组DNS工具包,里面有一些DNS相关的工具.主要有:dig,host,nslookup,nsupdate.使用这些工具可以进行域名解析和DNS调试工作。
 vim /etc/resolv.conf
  	#nameserver 127.0.0.1 #使用自己本机解析
 nslookup www.yutao.co 
 dig www.yutao.co #可以看到已经可以给自己进行域名解析
 vim /etc/named.conf #修改配置文件允许其他主机解析   
```
配置文件改为any
>  options {
>  
>         listen-on port 53 { any; };
>         
>         listen-on-v6 port 53 { any; };
>         
>         directory       "/var/named";
>         
>         dump-file       "/var/named/data/cache_dump.db";
>         
>         statistics-file "/var/named/data/named_stats.txt";
>         
>         memstatistics-file "/var/named/data/named_mem_stats.txt";
>         
>         recursing-file  "/var/named/data/named.recursing";
>         
>         secroots-file   "/var/named/data/named.secroots";
>         
>         allow-query     { any; };

```bash
service named restart #重启named服务
systemctl stop firewalled #关闭防火墙
systemctl status firewalled #查看到防火墙状态enable

```
客户机配置

```bash
 vim /etc/resolv.conf
    #namesever 192.168.2.4 IP地址是刚才的服务机
```
客户机使用服务机解析成功

2.主域名服务器
服务机
```bash
vim /etc/named.rfc1912.zones
```
添加配置告诉named为wyt.com提供域名解析


> zone "wyt.com" IN {
>        type master;
>         file "wyt.com.zone"; 
>        allow-update { none; }; };

file "wyt.com.zone";  # wyt.com.zone文件里存放DNS记录 A,CNAME,TXT

```bash
cd /var/named
cp  -a named.localhost wyt.com.cone # -a 保留原来属性，用户，组，时间，权限，因为named进程是name用户创建
vim wyt.com.zone 
```
添加一条A解析记录 www A 101.43.235.217
> $TTL 1D @       IN SOA  @ rname.invalid. (
>                                         0       ; serial
>                                         1D      ; refresh
>                                         1H      ; retry
>                                         1W      ; expire
>                                         3H )    ; minimum
>         NS      @
>         A       127.0.0.1
>         AAAA    ::1
>          www A 101.43.235.217

```bash
service named restart
```

> [root@localhost named]# nslookup www.wyt.com 
> Server:		127.0.0.1
> Address:	127.0.0.1#53
> Name:	www.wyt.com Address: 101.43.235.217

本机解析成功

客户机配置

```bash
 vim /etc/resolv.conf
    #namesever 192.168.2.4 IP地址是刚才的服务机
```

> [root@centos71 ~]# nslookup www.wyt.com 
> Server:		192.168.2.4
> Address:	192.168.2.4#53
> Name:	www.wyt.com Address: 101.43.235.217

客户机解析成功
