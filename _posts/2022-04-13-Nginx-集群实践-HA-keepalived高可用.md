---
layout: article
title: Nginx-集群实践-HA-keepalived高可用
tags: Nginx
category: blog
date: 2022-04-13 14:55:00 +08:00
mermaid: true
---


# 简介

 **高可用**：(英语：high availability)，缩写为 HA），指系统无中断地执行其功能的能力，代表系统的可用性程度，，防止单点故障，备份，高可用需要高成本。

# keepalived
**概念**：Keepalived的作用是检测服务器的状态，如果有一台web服务器宕机，或工作出现故障，Keepalived将检测到，并将有故障的服务器从系统中剔除，同时使用其他服务器代替该服务器的工作，当服务器工作正常后Keepalived自动将服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的服务器。

**核心技术**：VRRP协议和VIP

**漂移**： master宕机了，其它备份的服务器充当master服务器，这就称为漂移

**脑裂**： 两台或多台负载均衡器上都有VIP，这样的现象就是脑裂

**脑裂原因**：
- 高可用服务器对之间通信发生故障
- virtual_router_IP不一样
- 优先级相同的时候
- 开启防火墙
- 网卡及相关驱动坏了，ip配置及冲突
- 
**脑裂危害**：

 - 对运维人员而言，用户选中哪个VIP机器，运维人员是不可控的，那我们选择的master和backup 就没有任何的作用了。
 - 对用户而言没有害处，完全感受不到。
 - 对于后端反而还有负载均衡的作用。

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
![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/0e8cfe77-5588-4bc4-9cb0-d5e0ea954ba2)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/200fe20f-ba2e-4a4e-b0b9-22c01bfe5419)


![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/a57a14b7-6e97-4a6d-af03-6e1276fdd802)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/40439d17-08b3-4f4d-a8da-9a5e32da48f7)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/1217cb5b-6b0e-43a7-a36c-e0f7ca908026)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/6790b8dc-867e-46d1-8b83-e30063bc949d)

在192.168.2.188上关闭keepalived模拟宕机
```bash
service keepalived stop
```

可以看到当一台负载（192.168.2.188）发生故障，ip会漂移到另外一台，不影响用户使用，用户根本无法察觉。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/d74e60e3-4b69-4cc8-b514-cd086df50465)





