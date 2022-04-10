## SNAT代理内网上网
**SNAT**: （英语：Source Network Address Translation）**修改数据包的源地址**。
**应用环境**：局域网主机共享单个公网IP地址接入Internet
本实验中主机B的防火墙将数据包的源地址替换为192.168.2.188/24。这样使网络内部主机能够与网络外部通信。
**部署图**
![在这里插入图片描述](https://img-blog.csdnimg.cn/f2d2fd04fb99426eaae64b138bf1be69.png?客户机x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
**环境准备**
客户机A：(不能上网)
IP:192.168.70.1    系统CentOS 7.9
服务机B：(能上网，两块网卡ens33,ens37)
IP：
 LAN口192.168.70.254(ens37)
WAN口192.168.2.188 (ens33)  
    
  系统CentOS 7.9
  
  **部署过程**
   - 两台主机都关闭防火墙，桥接模式上网。
配置好对应的ip地址 。
- 配置客户机IP地址是192.168.70.1，网关是192.168.70.254

 -  服务机开启路由转发功能  。

```bash
#永久
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf 
sysctl -p   #加载生效
```
- 服务机iptables添加SNAT规则脚本

```bash
#!/bin/bash
iptables -F#清除防火墙规则
#添加snat策略
iptables -t nat -A POSTROUTING -s 192.168.70.0/24 -o ens33 -j SNAT --to-source 192.168.2.188
```
执行脚本
实验结果:客户机A可以 ping通百度上网

**开机自动加载SNAT规则**

```bash
[root@firewall ~]vim /etc/rc.local 
#添加一行
bash /root/snat.sh
[root@firewall ~]chmod +x /etc/rc.d/rc.local 
```

## DNAT-映射内网服务器
**DNAT**: （英语：Destination Network Address Translation）**修改数据包的目标IP地址**。

 **应用环境**:Internet中发布位于企业局域网内的服务器
 
 **部署图**
![在这里插入图片描述](https://img-blog.csdnimg.cn/7afa9562b30b4be1b74b64330e80bec3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
**环境准备**
IP:192.168.70.1    系统CentOS 7.9
服务机B：(能上网，两块网卡ens33,ens37)
IP：
  LAN口192.168.70.254(ens37)
	WAN口192.168.2.188 (ens33)  
    
   WEB客户机A：IP:192.168.70.1安装nginx
   MYSQL客户机C:IP:192.168.70.2安装mysql
  系统CentOS 7.9
  
**部署过程**
- 两台主机都关闭防火墙，桥接模式上网。
配置好对应的ip地址 。
- 配置WEB客户机A的IP地址是192.168.70.1，网关是192.168.70.254
- 配置MYSQL客户机C的IP地址是192.168.70.2，网关是192.168.70.254

-  **安装docker容器**
（docker又是一个DNAT原理）
```bash
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
 
```

```bash
cd /etc/yum.repos.d/
```

```bash
yum install docker-ce docker-ce-cli containerd.io -y
```

```bash
service docker start
```

```bash
docker pull nginx
```

```bash
docker run --name nginx-1 -d -p 80:80 nginx
#docker的80端口映射到主机的80端口
```

```bash
ss -anplut|grep docker#查看docker已经启动监听80端口
```

```bash
#修改nginx首页
docker exec -it nginx-1 /bin/bash
cd /usr/share/nginx/html
echo "wangyutao" >index.html
```

- 开启服务机的路由转发功能
- 添加使用DNAT策略的防火墙规则脚本

```bash
#!/bin/bash
#清除防火墙规则
iptables -F
#DNATWEB
iptables -t nat -A PREROUTING -i ens33 -d 192.168.2.188 -p tcp --dport 80 -j DNAT --to-destination 192.168.70.1:80
#DNATMYSQL
iptables -t nat -A PREROUTING -i ens33 -d 192.168.2.188 -p tcp --dport 3306 -j DNAT --to-destination 192.168.70.2:3306
```
执行脚本
```bash
#允许以admin用户远程连接数据库
grant all on *.* to admin@'%' identified by 'admin' with grant option;
flush privileges;
```

实验结果:
Windows主机输入190.168.2.188访问到容器里的nginx页面
![在这里插入图片描述](https://img-blog.csdnimg.cn/8f896d83656b4d14af957f6e524f4ef8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_16,color_FFFFFF,t_70,g_se,x_16)

Windows主机远程登录容器的数据库

![在这里插入图片描述](https://img-blog.csdnimg.cn/93aa0ca93e5e45b7bf2428f79d953fe5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

