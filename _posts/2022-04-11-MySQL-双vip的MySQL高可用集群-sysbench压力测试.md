---
layout: article
title: MySQL-双vip的MySQL高可用集群-sysbench压力测试
tags: MySQL数据库
category: blog
date: 2022-04-11 20:36:00 +08:00
mermaid: true
---

# 项目名称:双vip的MySQL高可用集群
## 项目环境：

7台服务器，centos7.8， mysql5.7.30，mysqlrouter8.0.23，keepalived2.0.10

## 项目描述：

本项目的目的是构建一个高可用的能实现读写分离的高效的MySQL集群,确保业务的稳定,能沟通方便的监控整个 集群,同时能批量的去部署和管理整个集群。

## 项目步骤:

 - 安装好centos7.8的系统，~~部署好ansible，在所有的机器之间配置SSH免密通道。~~ （后续添加）
 -  ~~部署好zabbix监控系统。~~ (后续添加)
- ~~通过ansible~~ 以二进制方式安装部署MySQL，主要是通过编写好的脚本一键安装二进制版本的MySQL。 
-  ~~使用ansible~~ 安装mysqlrouter和keepalived，在另外两台中间件服务器上，实现读写分离和高可用,在keepalived 配置2个实例，实现2个vip,互为master和backup，更加好的提升高可用的性能。 
- 在3台MySQL服务器上配置好主从复制,建立读写分离使用的用户，形成一个master+2个slave节点的集群,提供数据库服务，部署一台延迟备份的服务器（延迟60分钟）
- ~~尝试部署mysql的failover插件，实现自动的故障切换，确保naster宕机能自动提升另外一台slave为主，另外一台slave切换到新的mater上获得二进制日志。~~ 

- ~~验证测试读写分离和高可用以及主从的failover|~~ 

- 使用压力测试软件(sysbench)测试整个MySQL集群的性能（并发性能指标QPS、TPS、IOPS）

## 实际情况：

主机性能有限，所学知识有限，我这里使用了5台Centos服务器，其中两台中间件服务器，一台master，两台slave，一台Ubuntu服务器用于y压力测试检验(centos也行，只是我刚好剩一台Ubuntu)。

# 详细步骤
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/31a3f752c6694e8a99c49b25d8e1e39d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

 ## GTID主从复制
 - 上篇文章，我已经配置好了后端master(192.168.2.58)，slave1(192.168.2.218)，slave2(192.168.2.228)的GTID主从复制[点击跳转](https://blog.csdn.net/weixin_46415378/article/details/124049260?spm=1001.2014.3001.5501)

 
## 使用mysqlrouter配置读写分离
node1和node2配置一样
	
```bash
 wget https://repo.mysql.com/yum/mysql-tools-community/el/7/x86_64/mysql-router-community-8.0.23-1.el7.x86_64.rpm #下载mysqlrouter8.0.23的rpm包
 rpm -ivh mysql-router-community-8.0.23-1.el7.x86_64.rpm  #rpm安装mysqlrouter8.0.23
 vim /etc/mysqlrouter/mysqlrouter.conf #编辑配置文件追加如下内容
```

```bash
[routing:read_write]
bind_address = 0.0.0.0 
bind_port = 7001
mode = read-write
destinations = 192.168.2.58:3306
max_connections = 65535
max_connect_errors = 100
client_connect_timeout = 9

[routing:read_only]
bind_address = 0.0.0.0
bind_port = 7002
mode = read-only
destinations = 192.168.2.218:3306, 192.168.2.228:3306,192.168.2.58:3306
max_connections = 65535
max_connect_errors = 100
client_connect_timeout = 9
```

```bash
service mysqlrouter start #启动mysqlrouter
ss -anplut #查看mysqlrouter开启监听到7001端口 7002端口情况
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/18ded2defc3f4a758487f41618028959.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**注意**：关闭防火墙和SELINUX

```bash
setenforce 0 #临时关闭
vi /etc/selinux/config 
#将SELINUX=enforcing改为SELINUX=disabled，永久关闭
```
```bash
service  firewalld stop #关闭防火墙
systemctl disable firewalld # 永久关闭
```

 **master授权读写分离用户**

```bash
create user 'wangyutao'@'%' identified by '123456';
grant all on *.* to 'wangyutao'@'%';
#读写用户wangyutao
create user 'yutao'@'%' identified by '123456';
grant select on *.* to 'yutao'@'%';
#只读用户yutao
```
	
	

> 192.168.2.188:7001 --->wangyutao---> read_write
> 	192.168.2.188:7002--->yutao--->read_only
> 	
> 	192.168.2.108:7001 --->wangyutao---> read_write
> 	192.168.2.108:7002--->yutao--->read_only

	
**使用安装了MySQL的Windows主机检验**：

	![在这里插入图片描述](https://img-blog.csdnimg.cn/e8c85fb74ea74ab7a45efc50dbd44273.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
	
   > 可读写

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a2a4148a2284c6181329d32927500da.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
	

  > 只能读，不能写

	
检验node2同理（图片不展示，避免篇幅过长，最后展示最终结果）

## 配置双vip，在中间件上配置keepalived
两台中间件node服务器都安装好keepalived

	

```bash
yum install keepalived -y
```

**node1配置**

```bash
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
#   vrrp_strict  注意注释 vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER
interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    mcast_src_ip 192.168.2.188
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.2.30
     }
}


vrrp_instance VI_2 {
    state BACKUP
    interface ens33
    virtual_router_id 52
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.2.31
    }
}
```
ip add命令查看到虚拟IP地址192.168.2.30
![在这里插入图片描述](https://img-blog.csdnimg.cn/32b8d6e8e27e4e1786ae7927c7dbfa36.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

node2配置

```bash
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
#   vrrp_strict   注意注释 vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state BACKUP
interface ens33
    virtual_router_id 51
    priority 90
    advert_int 1
    mcast_src_ip 192.168.2.108
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }   
    virtual_ipaddress {
        192.168.2.30
     }  
}    

vrrp_instance VI_2 {
    state MASTER
    interface ens33
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.2.31
    }
}
```
ip add命令查看到虚拟IP地址192.168.2.31
![在这里插入图片描述](https://img-blog.csdnimg.cn/ad674e1ab5154a8f9d50f7c2951e22e3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

## 验证双vip是否能在master和backup上漂移

在node1上停止Keepalived服务
![在这里插入图片描述](https://img-blog.csdnimg.cn/b72a3741afbe4a09a9c6d144f2786ad2.png)
node1 192.168.2.30 漂移到了node2
![在这里插入图片描述](https://img-blog.csdnimg.cn/ea1792cc38184a77850846f75e6e76ba.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
开启node1keepalived服务后，又漂移回去。

## Windows检验通过keepalived高可用vip操作数据库

![在这里插入图片描述](https://img-blog.csdnimg.cn/f19610566604437794fbe7178302be64.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/97cd3431bffd4f9fb23b37ef2807c049.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/db24c593bac24b4f95ce1b77261e2000.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a8e36e4b1c66467e89403e8ce2353535.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

## sysbench压力测试
使用Ubuntu服务器

```bash
apt install sysbench -y #安装sysbench压力测试工具
```
master 建库
```bash
mysql>create database sbtest 
#建立这个库运行usr/share/sysbench下的oltp_read_write.lua脚本压测需要先建立这个库
```
Ubuntu服务器进行压测
```bash
sysbench --threads=64 --time=600 --histogram=on --mysql_host=192.168.2.30 --mysql-port=7001 --mysql-user=wangyutao --mysql-password=123456 /usr/share/sysbench/oltp_read_write.lua --table_size=1000 prepare 
#准备工作，建表，造数据
sysbench --threads=64 --time=600 --histogram=on --mysql_host=192.168.2.30 --mysql-port=7006 --mysql-user=wangyutao --mysql-password=123456 /usr/share/sysbench/oltp_read_write.lua --table_size=1000 run 
#启动压测程序
```
**top命令实时对系统处理器的状态监视**

![在这里插入图片描述](https://img-blog.csdnimg.cn/9d1bdad2b2274e298d8cbf549f425906.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/34ae6001477d49edab85e1d509107258.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
可以看到master的mysqld进程，node的mysqlrouter进程cpu使用率突升

压力测试后考虑下优化思路

## MySQL优化思路

- 升级硬件
- 优化操作系统：优化linux的内核参数，网络参数，文件系统的参数
- 优化MySQL的参数：例如buffer-pool的大小等其他参数
- 优化SQL语句
- 加缓冲，加中间件，读写分离
- 分表分库
- 加索引

**垂直切分**

库：将一台机器的库，分到多个机器
表：将表的字段拆分（一个表垂直切开）

**水平切分**

库：将多个表分散到多个库中
表：不同的行分到不同的表。

# 项目心得

-  一定要规划好整个集群的架构,配置要细心,脚本要提前准备好,边做边修改。

-  防火墙和selinux的问题需要多注意。

-  对MySQL的集群和高可用有了深入的理解。

- 对自动化批量部署和监控有了更加多的应用和理解。

-  keepalived的配置需要更加细心和IP地址的规划有了新的认识。 

- 对双vip的使用,我们可用考虑在前面加一个负载均衡器或者使用dns轮询。
