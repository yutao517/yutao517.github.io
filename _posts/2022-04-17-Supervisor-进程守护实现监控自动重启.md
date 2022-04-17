---
layout: article
title: Supervisor-进程守护实现监控自动重启
tags: Supervisor
category: blog
date: 2022-04-17 15:55:00 +08:00
mermaid: true
---

## 简介

**Supervisor**是一个基于Python开发的Linux系统上的进程监控工具
管理和监控Linux上面的进程，可以很方便的监听、启动、停止和重启一个或多个进程。通过 Supervisor 管理的进程，当进程意外被 Kill 时，Supervisor 会自动将它重启，可以很方便地做到进程自动恢复的目的，而无需自己编写 shell 脚本来管理进程。

## 原理
在supervisor的配置⽂件中，把要管理的进程的可执行文件的路径写进去，通过配置command这个参数，把这些被管理的进程当作supervisor的子进程来启动，获取到该进程的pid，然后再对该pid进行监控，当子进程挂掉的时候，父进程可以准确获取子进程挂掉的信息，可以选择是否⾃⼰启动和报警。
## 安装 Supervisor
CentOS安装
```bash
yum install -y epel-release  #安装epel源
yum install -y supervisor    #安装supervisor
systemctl enable supervisord # 开机自启动
systemctl start supervisord  # 启动supervisord服务
systemctl status supervisord # 查看supervisord服务状态
ps -aux|grep supervisord     # 查看是否存在supervisord进程
vim /etc/supervisord.conf    #编辑配置文件
```

```bash
[inet_http_server]        #HTTP服务器，提供web管理界面
port=0.0.0.0:9001         #Web管理后台运行的IP和端口，云服务器注意开放该端口
username=admin            #登录管理后台的用户名
password=admin            #登录管理后台的密码
```
```bash
cd /etc/supervisord.d/
vim wyt.ini             #自定义.ini文件的名字
```

```bash
[program:api]
directory=/root/
#执行文件的路径
command=bash test.sh
#执行的命令
autostart = true
#随supervisor启动
startsecs = 10
#启动10秒后没有异常退出，就表示进程正常启动
autorestart = true
#程序退出后自动重启
startretries = 2
#启动失败自动重试次数
user = root
#执行命令的用户
stopsignal=KILL               
#用来杀死进程的信号
stdout_logfile=/root/tornado_16018.log
#日志路径
```

```bash
systemctl start supervisord 
```

浏览器访问IP地址端口
>  http://0.0.0.0:9001  

![在这里插入图片描述](https://img-blog.csdnimg.cn/241d2133635d46299843874f88236995.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
已经开始运行并监控，如果进程挂掉将重启，但是supervisor不支持跨机器的进程监控，一个supervisord只能监控本机上的程序，大大限制了supervisor的使用。

## 常用命令
- supervisorctl status 查看进程运行状态

- supervisorctl start 进程名 启动进程

- supervisorctl stop 进程名 关闭进程

- supervisorctl restart 进程名 重启进程

- supervisorctl update 	重新载入配置文件

- supervisorctl shutdown 关闭supervisord

- supervisorctl clear 进程名 清空进程日志

- start stop restart + all 表示启动，关闭，重启所有进程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/1b5db299f2024b32a6a8fc9bdaf89e2b.png)
