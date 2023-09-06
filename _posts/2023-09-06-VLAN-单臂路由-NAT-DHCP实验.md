---
layout: article
title: VLAN-单臂路由-NAT-DHCP实验
tags: 计算机网络
category: blog
date: 2023-09-06 00:00:00 +08:00
mermaid: true
---
拓扑

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/957c9041-609f-4db1-8768-9946fb07b040)


PC1

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/fc106d77-ed17-4366-bddd-654c3fc6184b)


PC2

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/05028cbd-7656-4817-9605-2bdf0090ad6e)

GW

打开接口
```bash
Router#en
Router#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  administratively down down    
FastEthernet1/0            unassigned      YES unset  administratively down down    
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f0/0
Router(config-if)#no sh
*Mar  1 00:30:48.223: %LINK-3-UPDOWN: Interface FastEthernet0/0, changed state to up
*Mar  1 00:30:49.223: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up
Router(config)#exit
Router#
*Mar  1 00:30:53.767: %SYS-5-CONFIG_I: Configured from console by console
Router#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up      
FastEthernet1/0            unassigned      YES unset  administratively down down    
Router#
```
SW

```bash
Router>en
Router#vlan data
Router#vlan database 
Router(vlan)#vlan 10
VLAN 10 added:
    Name: VLAN0010
Router(vlan)#vlan 20
VLAN 20 added:
    Name: VLAN0020
Router(vlan)#exit
APPLY completed.
Exiting....
Router#show vlan-sw
Router#show vlan-switch bri
Router#show vlan-switch brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/0, Fa0/1, Fa0/2, Fa0/3
                                                Fa0/4, Fa0/5, Fa0/6, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa1/0, Fa1/1, Fa1/2, Fa1/3
                                                Fa1/4, Fa1/5, Fa1/6, Fa1/7
                                                Fa1/8, Fa1/9, Fa1/10, Fa1/11
                                                Fa1/12, Fa1/13, Fa1/14, Fa1/15
10   VLAN0010                         active    
20   VLAN0020                         active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
Router#
```

```bash
Router#conf t  
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f0/1
Router(config-if)#switchport mode acc
Router(config-if)#switchport mode access 
Router(config-if)#switchport acc
Router(config-if)#switchport access vlan 10
Router(config-if)#exit
Router(config)#int f0/2
Router(config-if)#switchport mode access   
Router(config-if)#switchport access vlan 20
Router(config-if)#exit
Router(config)#exit
Router#sho
*Mar  1 00:38:31.515: %SYS-5-CONFIG_I: Configured from console by conso   
% Type "show ?" for a list of subcommands
Router#
Router#show vlan-sw
Router#show vlan-switch bri
Router#show vlan-switch brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/0, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa1/0, Fa1/1
                                                Fa1/2, Fa1/3, Fa1/4, Fa1/5
                                                Fa1/6, Fa1/7, Fa1/8, Fa1/9
                                                Fa1/10, Fa1/11, Fa1/12, Fa1/13
                                                Fa1/14, Fa1/15
10   VLAN0010                         active    Fa0/1
20   VLAN0020                         active    Fa0/2
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    

```

```bash
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f0/0
Router(config-if)#switch
Router(config-if)#switchport tr
Router(config-if)#switchport trunk en
Router(config-if)#switchport trunk encapsulation do
Router(config-if)#switchport trunk encapsulation dot1q 
Router(config-if)#switc
Router(config-if)#switchport mode tr
Router(config-if)#switchport mode trunk 
Router(config-if)#
*Mar  1 00:44:01.579: %DTP-5-TRUNKPORTON: Port Fa0/0 has become dot1q trunk
Router(config-if)#end
Router#show
*Mar  1 00:44:12.323: %SYS-5-CONFIG_I: Configured from console by console 
% Type "show ?" for a list of subcommands
Router#show int tr      
Router#show int trunk 

Port      Mode         Encapsulation  Status        Native vlan
Fa0/0     on           802.1q         trunking      1

Port      Vlans allowed on trunk
Fa0/0     1-1005

Port      Vlans allowed and active in management domain
Fa0/0     1,10,20

Port      Vlans in spanning tree forwarding state and not pruned
Fa0/0     1,10,20
Router#show vlan-sw
Router#show vlan-switch br
Router#show vlan-switch brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa1/0, Fa1/1, Fa1/2
                                                Fa1/3, Fa1/4, Fa1/5, Fa1/6
                                                Fa1/7, Fa1/8, Fa1/9, Fa1/10
                                                Fa1/11, Fa1/12, Fa1/13, Fa1/14
                                                Fa1/15
10   VLAN0010                         active    Fa0/1
20   VLAN0020                         active    Fa0/2
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    

```
GW
```bash
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f0/0
Router(config-if)#no sh
Router(config-if)#exit
Router(config)#int f0/0.?
  <0-4294967295>  FastEthernet interface number

Router(config)#int f0/0.10  
Router(config-subif)#exit
Router(config)#int f0/0.20
Router(config-subif)#end
Router#sj
*Mar  1 00:58:33.155: %SYS-5-CONFIG_I: Configured from console by consol 
% Type "show ?" for a list of subcommands
Router#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up      
FastEthernet0/0.10         unassigned      YES unset  up                    up      
FastEthernet0/0.20         unassigned      YES unset  up                    up      
FastEthernet1/0            unassigned      YES unset  administratively down down    
Router#en 
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f0/0.10
Router(config-subif)#en
Router(config-subif)#encapsulation do
Router(config-subif)#encapsulation dot1Q 10    ##服务vlan10
Router(config-subif)#ip add 192.168.1.1 255.255.255.0   ##设置网关IP
Router(config-subif)#exit
Router(config)#int f0/0.20                     
Router(config-subif)#encapsulation dot1Q 20          
Router(config-subif)#ip add 192.168.2.1 255.255.255.0
Router(config-subif)#exit

```


PC1的测试

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/b22b508d-34be-4700-a9ee-7b8ce7c2e422)


交换机可以转发广播包，路由器不可以转发广播包。
实验结果，PC1、PC2通过路由器互通，且VLAN隔离


GW
```bash
Router>
Router>
Router>
Router>en 
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f1/0
Router(config-if)#ip add 100.1.1.2 255.255.255.0
Router(config-if)#no sh 
Router(config-if)#exit
Router(config)#
*Mar  1 01:14:37.587: %LINK-3-UPDOWN: Interface FastEthernet1/0, changed state to up
*Mar  1 01:14:38.587: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet1/0, changed state to up 
Router(config)#exit
Router#show 
*Mar  1 01:14:56.795: %SYS-5-CONFIG_I: Configured from console by console
% Type "show ?" for a list of subcommands
Router#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up      
FastEthernet0/0.10         192.168.1.1     YES manual up                    up      
FastEthernet0/0.20         192.168.2.1     YES manual up                    up      
FastEthernet1/0            100.1.1.2       YES manual up                    up      
Router#
```
ISP
```bash
Router#show ip int br
*Mar  1 01:17:43.563: %SYS-5-CONFIG_I: Configured from console by console
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  administratively down down    
Router#conf t        
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f0/0
Router(config-if)#ip add 100.1.1.1 255.255.255.0
Router(config-if)#int 
Router(config-if)#exit
Router(config)#int
Router(config)#interface loo
Router(config)#interface loopback 0
Router(config-if)#
*Mar  1 01:18:41.687: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
Router(config-if)#ip add 8.8.8.8 255.255.255.0
Router(config-if)#end
Router#
*Mar  1 01:19:23.227: %SYS-5-CONFIG_I: Configured from console by console    
Router#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            100.1.1.1       YES manual administratively down down    
Loopback0                  8.8.8.8         YES manual up                    up      
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f0/0
Router(config-if)#no sh
Router(config-if)#end
Router#
```



```
1. ACL permit 抓取数据
2. 转换数据--100.1.1.2  连接运营商的外网地址  --f1/0
3. 定义内部接口和外部接口

通过一个公网地址的多个端口号来区分不同的内部业务数据---overload端口复用

GW
```bash
Router>en
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#
Router(config)#ip route 0.0.0.0 0.0.0.0 100.1.1.1
Router(config)#access
Router(config)#access-list 1 p
Router(config)#access-list 1 permit 192.168.1.0 0.0.0.255
Router(config)#access-list 1 permit 192.168.2.0 0.0.0.255
Router(config)#ip nat in
Router(config)#ip nat inside sou
Router(config)#ip nat inside source list 1 int
Router(config)#ip nat inside source list 1 interface f1/0 over
Router(config)#ip nat inside source list 1 interface f1/0 overload 
Router(config)#
*Mar  1 01:47:33.431: %LINEPROTO-5-UPDOWN: Line protocol on Interface NVI0, changed state to up
Router(config)#
```

```bash
Router>en      
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f0/0
Router(config-if)#ip nat ins
Router(config-if)#ip nat inside 
Router(config-if)#exit
Router(config)#end  
Router#show
*Mar  1 01:58:51.155: %SYS-5-CONFIG_I: Configured from console by console 
% Type "show ?" for a list of subcommands
Router#show ip inter br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up      
FastEthernet0/0.10         192.168.1.1     YES manual up                    up      
FastEthernet0/0.20         192.168.2.1     YES manual up                    up      
FastEthernet1/0            100.1.1.2       YES manual up                    up      
NVI0                       unassigned      NO  unset  up                    up      
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f1/0
Router(config-if)#ip nat outside
Router(config-if)#exit
```

```bash
Router>en      
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f0/0
Router(config-if)#ip nat ins
Router(config-if)#ip nat inside 
Router(config-if)#exit
Router(config)#end  
Router#show
*Mar  1 01:58:51.155: %SYS-5-CONFIG_I: Configured from console by console 
% Type "show ?" for a list of subcommands
Router#show ip inter br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up      
FastEthernet0/0.10         192.168.1.1     YES manual up                    up      
FastEthernet0/0.20         192.168.2.1     YES manual up                    up      
FastEthernet1/0            100.1.1.2       YES manual up                    up      
NVI0                       unassigned      NO  unset  up                    up      
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#int f1/0
Router(config-if)#ip nat outside
Router(config-if)#exit
Router(config)#int f0/0
Router(config-if)#no ip nat in
Router(config-if)#no ip nat inside 
Router(config-if)#exit
Router(config)#int f0/0.10
Router(config-subif)#ip nat inside
Router(config-subif)#exit
Router(config)#int f0/0.20  
Router(config-subif)#ip nat inside
Router(config-subif)#exit  
```
inside必须定义到有IP的接口上，子接口。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/71dc40bb-1c88-404a-bde1-9ba89dcc60d9)


当ISP新增环回接口模拟新的服务4.4.4.4，PC1和PC2依然能访问。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/74b0daad-082d-49bc-85dd-ff3c1060bee8)

假如需要添加公网地址，需要定义公网池
```bash
Router(config)#no ip nat inside source list 1 interface f1/0 overload 
Router(config)#ip nat pool CCNA
% Incomplete command.

Router(config)#ip nat pool CCNA ?
  A.B.C.D        Start IP address
  netmask        Specify the network mask
  prefix-length  Specify the prefix length

Router(config)#ip nat pool CCNA 100.1.1.2 100.1.1.3 net
Router(config)#ip nat pool CCNA 100.1.1.2 100.1.1.3 netmask 255.255.255.0
Router(config)#ip nat inside source list 1 pool CCNA overload 
```
当100.1.1.2端口用满之后，就会使用100.1.1.3

----


静态nat

```bash
#取消动态nat
gw#conf t
gw(config)#interface f1/0
gw(config-if)#no ip nat outside
gw(config-if)#exit
gw(config)#int f0/0.10
gw(config-subif)#no ip nat in  
gw(config-subif)#exit        
gw(config)#int f0/0.20 
gw(config-subif)#no ip nat inside
gw(config-subif)#end
gw(config)#no access-list 1
gw(config)#no ip nat pool CCNA
gw(config)#exit
gw#show run | in ip nat


#开启静态nat
gw(config)#interface f1/0
gw(config-if)#ip nat outside
gw(config-if)#exit
gw(config)#int f0/0.10
gw(config-subif)#ip nat inside  
gw(config-subif)#exit        
gw(config)#int f0/0.20 
gw(config-subif)#ip nat inside
gw(config-subif)#exit
gw(config)#ip nat inside source static tcp 192.168.1.10 23 100.1.1.2 1000   
gw(config)#ip nat inside source static tcp 192.168.2.10 23 100.1.1.2 2000
gw(config)#exit

##打开telnet
gw(config)#line vty 0 4
gw(config-line)#password cisco
gw(config-line)#login
gw(config-line)#exit

pc1(config)#line vty 0 4
pc1(config-line)#password cisco
pc1(config-line)#login
pc1(config-line)#end

pc2(config)#line vty 0 4
pc2(config-line)#password cisco
pc2(config-line)#login
pc2(config-line)#end
```

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/19e4eabb-1553-44d2-9b47-fdf073ff4d04)


DHCP 动态IP配置
```bash![请添加图片描述](https://img-blog.csdnimg.cn/71e8eecbfe044d1f9679a1f68e2c4222.png)

gw#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
gw(config)#
gw(config)#
gw(config)#
gw(config)#ip dhcp pool ?   
  WORD  Pool name

gw(config)#ip dhcp pool DHCP
gw(dhcp-config)#
gw(dhcp-config)#netwo
gw(dhcp-config)#network 192.168.1.0 255.255.255.0 
gw(dhcp-config)#defa
gw(dhcp-config)#default-router 192.168.1.1 
gw(dhcp-config)#dns-
gw(dhcp-config)#dns-server 8.8.8.8
gw(dhcp-config)#
gw(dhcp-config)#end 
gw#
*Mar  1 06:44:53.478: %SYS-5-CONFIG_I: Configured from console by console     
gw#show run | sec dhcp
no ip dhcp use vrf connected
ip dhcp pool DHCP
   network 192.168.1.0 255.255.255.0
   default-router 192.168.1.1 
   dns-server 8.8.8.8 


gw#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
gw(config)#ip dhcp poo
gw(config)#ip dhcp pool DHCP-2
gw(dhcp-config)#net
gw(dhcp-config)#netw
gw(dhcp-config)#network 192.168.2.0 /24
gw(dhcp-config)#defa
gw(dhcp-config)#default-router 192.168.2.1
gw(dhcp-config)#dns-se
gw(dhcp-config)#dns-server 4.4.4.4
gw(dhcp-config)#end
gw#
*Mar  1 06:47:48.830: %SYS-5-CONFIG_I: Configured from console by console
gw#

```

PC1
```bash
pc1>en 
pc1#
pc1#
pc1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
pc1(config)#
pc1(config)#int f0/0
pc1(config-if)#no ip add
pc1(config-if)#ip add dh
pc1(config-if)#ip add dhcp ?
  client-id  Specify client-id to use
  hostname   Specify value for hostname option
  <cr>

pc1(config-if)#ip addr     
pc1(config-if)#ip address dhcp
pc1(config-if)#exit

pc1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.1.2     YES DHCP   up                    up      
NVI0                       unassigned      NO  unset  up                    up      

gw(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.100 ##设置不允许下发192.168.1.1到192.168.1.100的地址
pc1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
pc1(config)#int f0/0
pc1(config-if)#sh
pc1(config-if)#no sh
pc1(config-if)#
*Mar  1 07:05:40.634: %LINK-5-CHANGED: Interface FastEthernet0/0, changed state to administratively down
*Mar  1 07:05:41.634: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to down
pc1(config-if)#
*Mar  1 07:05:44.222: %LINK-3-UPDOWN: Interface FastEthernet0/0, changed state to up
*Mar  1 07:05:45.222: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up
pc1(config-if)#end
pc1#
*Mar  1 07:05:48.778: %DHCP-6-ADDRESS_ASSIGN: Interface FastEthernet0/0 assigned DHCP address 192.168.1.101, mask 255.255.255.0, hostname pc1

*Mar  1 07:05:50.210: %SYS-5-CONFIG_I: Configured from console by console
pc1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.1.101   YES DHCP   up                    up      
NVI0                       unassigned      NO  unset  up                    up      
pc1#  
```

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/8e4a8fb3-3477-4144-bbf0-b530f632e642)
