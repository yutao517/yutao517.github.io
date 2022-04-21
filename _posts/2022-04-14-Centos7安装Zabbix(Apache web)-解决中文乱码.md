---
layout: article
title: Centos7安装Zabbix(Apache web)-解决中文乱码
tags: Zabbix
category: blog
date: 2022-04-14 12:16:00 +08:00
mermaid: true
---

## 概念
Zabbix 是由 Alexei Vladishev 开发的一种网络监视、管理系统，基于 Server-Client 架构。可用于监视各种网络服务、服务器和网络机器等状态。

[Zabbix官方手册](https://www.zabbix.com/documentation/current/zh/manual)
## 常见的监控软件
- nagios
- cacti
- zabbix
- prometheus
- netdata
- open-falcon

## 安装
[官方文档](https://www.zabbix.com/cn/download?zabbix=5.0&os_distribution=centos&os_version=7&db=mysql&ws=apache)

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
#安装Zabbix存储库
yum install zabbix-server-mysql zabbix-agent
#安装Zabbix服务器和代理
yum install centos-release-scl
#启用红帽软件集合
vi /etc/yum.repos.d/zabbix.repo 
#打开前端功能
[zabbix-frontend]
...
enabled=1
...
yum install zabbix-web-mysql-scl zabbix-apache-conf-scl
#安装 Zabbix前端apache包
yum install mariadb mariadb-server -y
#安装mariadb数据库
service mariadb start #打开mariadb
systemctl enable mariadb #mariadb开机自启动
```

```bash
mysql -uroot -p
#直接回车，不输入密码，mariadb安装好没有密码
mysql>create database zabbix character set utf8 collate utf8_bin;
mysql>create user zabbix@localhost identified by '123456';
mysql>grant all privileges on zabbix.* to zabbix@localhost;
mysql>quit;
```

```bash
cd /usr/share/doc/zabbix-server-mysql-5.0.22/
gunzip create.sql.gz
cat create.sql |mysql -uzabbix -p zabbix
#输入密码123456
```

```bash
vim /etc/zabbix/zabbix_server.conf 
:124
DBPassword=123456 #解除注释,密码设置为123456
vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf 
php_value[date.timezone] = Asia/Shanghai
#最后一行;去掉，时区改为Asia/Shanghai
```

```bash
service firewalld stop
systemctl disable firewalld
setenforce 0
vim /etc/sysconfig/selinux
#改SELINUX=disabled
```

```bash
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
#启动Zabbix server和agent进程,设置开机自启
ps aux|grep zabbix
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/283ffa9877544ced85c0caccddedc385.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

> 浏览器输入你自己机器的IP地址/zabbix/
> http://192.168.2.138/zabbix/setup.php
> 
next step 
都是OK 
![在这里插入图片描述](https://img-blog.csdnimg.cn/96ae295890204539993ad426f962824b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
next step 
密码输入刚才设置的123456
![在这里插入图片描述](https://img-blog.csdnimg.cn/00f5f5b64562415fa5d7e457ea7ccea2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
next step
host:zabbix-server的IP地址
name:zabbix-server
server的端口号10051(agent 10050)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a8fb9e79cb54415d91370db897812382.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
恭喜你安装成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/e1dc12be273548e49bff86a00b7e7ffd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c4557b50402d466cb2bf24121dbbe402.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_14,color_FFFFFF,t_70,g_se,x_16)
> 默认的用户名：Admin 默认的密码：zabbix

```
mysql>use zabbix;
mysql>select * from users\G;
mysql>update users set passwd=md5('zabbix') where userid='1';
#可以修改初始密码
```

User setings
设置语言为中文
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ef901dcb88d425e9c0cb3671105480c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**解决图形中文乱码**

- 从Windows里面C盘->Windows->fonts 复制楷体常规(simkai.ttf)字体到**桌面**。
- 再用xftp传到zabbix-server服务器的/usr/share/zabbix/assets/fonts/字体目录下。(不复制到桌面无法传输)

 ```bash
vim /usr/share/zabbix/include/defines.inc.php 
:81 
#定位到81行，把默认字体改为simkai
:wq #退出并保存
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2d3ad11860dc420cbd484567b56bf5b2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
可以正常显示中文啦
![在这里插入图片描述](https://img-blog.csdnimg.cn/db297d1280a94eb6948c9e6f4d89abd8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)


