---
layout: article
title: Nginx-集群实践-HA-keepalived高可用
tags: Nginx
category: blog
date: 2022-04-13 14:55:00 +08:00
mermaid: true
---


# 简介

 **高可用**：(英语：high availability)，缩写为 HA），指系统无中断地执行其功能的能力，代表系统的可用性程度，高可用需要高成本，防止单点故障，备份。

# keepalived
**概念**：Keepalived的作用是检测服务器的状态，如果有一台web服务器宕机，或工作出现故障，Keepalived将检测到，并将有故障的服务器从系统中剔除，同时使用其他服务器代替该服务器的工作，当服务器工作正常后Keepalived自动将服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的服务器。

**核心技术**：VRRP协议和VIP

## VRRP协议
**虚拟路由器冗余协议**(英语：Virtual Router Redundancy Protocol，缩写为 VRRP)：它保证当主机的下一跳路由器坏掉时，可以及时由另一台路由器来替代，从而保证通讯的连续性和可靠性。

**VRRP工作原理**
**Master路由的选举**

VRRP协议的状态共有三种，Initialize，Master，Backup，初始状态都是Initialize，通过比较优先级产生Maste和Backup。VRRP根据优先级确定自己在备份组中的角色。优先级高的路由器成为Master路由器，优先级低的成为Backup路由器。Master路由器定期发送VRRP通告报文，通知备份组内的其他路由器自己工作正常；Backup路由器则启动定时器等待通告报文的到来，如果在一定时间内没有收到VRRP报文，则路由器切换为Master状态。

## HA的实验
五台centos服务器，两台负载均衡（192.168.2.108 、192.168.2.188），三台后端（192.168.2.58、192.168.2.218、192.168.228）

负载均衡器，两台配置一样。
192.168.2.188 192.168.2.108

```bash
vim /etc/usr/local/nginx/conf/nginx.conf
```

```bash
 http {
 	upstream myapp1 {
        server 192.168.2.58;
        server 192.168.2.218;
        server 192.168.2.228;
    }
	 server {
        listen 80;

        location / {
        	# root   html; 需要注释
            # index  index.html index.htm; 需要注释
            proxy_pass http://myapp1;
        }
    }
}
```

## keepaliived配置
**互为主备**

**192.168.2.188的keepalived配置**

```bash
vim /etc/keepalived/keepalived.conf
```

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
#   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
vrrp_instance VI_1 { #启动一个实例
    state MASTER #角色
    interface ens33 #网络接口
    virtual_router_id 51 #虚拟路由id,0-255
    priority 100 #优先级
    advert_int 1  #宣告消息间隔1s
    mcast_src_ip 192.168.2.188 #本机ip
    nopreempt
    authentication { 认证
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
**192.168.2.108配置**


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
#   vrrp_strict
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

结果，访问192.168.2.31或者192.168.2.30产生轮询效果，其中一台负载均衡器发生故障可以漂移到另外一台备用机不影响使用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/58528780b55e4adf9a12fc9380a999fb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4b25e3e9179e4f10a59ff5dab14ab41e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4fb881e6e0cf4bf4b48d9197b50cf3eb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/544543a553e0424d9af5b9266f09d6a3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9910c89e039e4e399b4a81e2b37e9b04.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d3d09b15d0de4ee4a42918670f84f10a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
在192.168.2.188上关闭keepalived模拟宕机
```bash
service keepalived stop
```

可以看到当一台负载（192.168.2.188）发生故障，ip会漂移到另外一台，不影响用户使用，用户根本无法察觉。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a2d0c58375c14246944394a224047194.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)




