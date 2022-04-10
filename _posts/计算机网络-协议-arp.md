![在这里插入图片描述](https://img-blog.csdnimg.cn/ab671bb49188420da67364863fa64d1a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_15,color_FFFFFF,t_70,g_se,x_16)
## 应用层协议
**超文本传输协议**(英语：HyperText Transfer Protocol，缩写**HTTP**)
**超文本传输安全协议**（英语：HyperText Transfer Protocol Secure，缩写：**HTTPS**）
**文件传输协议**（英语：File Transfer Protocol，缩写：**FTP**）
**简单文件传输协议**（英语Trivial File Transfer Protocol, 缩写**TFTP**）
**域名解析协议**（英语：Domain Name System，缩写：**DNS**）
**简单邮递发送协议**（英语：Simple Mail Transfer Protocol，缩写：**SMTP**）
**简单网络管理协议**（英语：Simple Network Management Protocol，缩写：**SNMP**）
**安全外壳协议**（英语：Secure Shell，缩写：**SSH**）

## 网络层协议

**地址解析协议**（英语：Address Resolution Protocol，缩写：**ARP**）通过解析网络层地址来找寻数据链路层地址的网络传输协议就是通过IP地址来定位MAC地址

**ARP欺骗**
原理：伪造ARP广播包，攻击者发送假的ARP数据包到网络上，尤其是送到网关上，目的是要让送至特定的IP地址的流量被错误送到攻击者所取代的地方。

防制方法：
1.静态绑定每台电脑的ARP。
2.抓包工具分析病毒源
3.安装arp防火墙

**互联网消息控制协议**（英语：Internet Control Message Protocol，缩写：**ICMP**）
在IP主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。
ICMP协议依赖于IP协议
“Ping”的过程实际上就是ICMP协议工作的过程
ping -s 数据包大小
ping -c 数量
ping -i 时间间隔
ping -w 超时时间
8   Echo request——回显请求
0   Echo reply——回显应答

## 经典面试题
如果ping 一台服务器ping不通可能有哪些原因？
分段排查自己的网络
1. Ping 127.0.0.1
如果本地循环地址无法Ping通，则表明本地机TCP/IP协议不能正常工作。
2. Ping本机的IP地址
用IPConfig查看本机IP，然后Ping该IP，通则表明网络适配器（网卡或MODEM）工作正常，不通则是网络适配器出现故障。 
3. Ping网关
如果Ping不通网关就是自己的配置或者路由出现问题。
4. Ping几个网址
比如ping baidu.com或者qq.com ping不通则是自己网络问题，或者运营商问题，通则Ping目标机房的服务器，不通初步判定是目标服务器的问题。
5. 服务器问题可能是挂了宕机，或者存在防火墙，可以访问服务器的网页是否能打开，能打开说明是防火墙，不是挂了。
