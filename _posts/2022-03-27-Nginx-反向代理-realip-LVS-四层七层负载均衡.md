---
layout: article
title: Nginx-反向代理-realip-LVS-四层七层负载均衡
tags: Nginx
category: blog
date: 2022-03-27 000000 +0800
mermaid: true
---

## 代理
 
**正向代理**：正向代理作用在客户端，我们访问一个网站比如只能在校园访问的校园网站，校外访问不了，但是我们可以通过一个代理服务器，比如一台在校内的服务器，校内服务器可以访问到校园网站，那么我们连上这个服务器，让它请求我们需要的网站，再返回给我们，这就是正向代理。

**反向代理**：反向代理是作用在服务器端的，对于用户的一个请求，会转发到多个后端处理器中的一台来处理该具体请求，但是用户并不知道自己的数据被转发到其他处理器。常见的DNS解析和负载均衡就是反向代理。

**Nginx**支持反向代理，可以做内容缓冲和负载均衡器。
CDN内容分发网络就使用了Nginx反向代理，缓存网站的静态内容。

**作用**：

1.在搭建VPN时，只要用到Nginx反向代理功能搭建，那么VPN速度会大大提高。原因就是Nginx反向代理做了内容缓冲。
2.负载均衡，解决容错，用户访问量过大，支持更高并发请求。

正向代理：是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，而后代理向原始服务器转交请求并将得到的内容返回给客户端。客户端必需要进行一些特别的设置才能使用正向代理。

正向代理的用途：一、访问原来没法访问的资源，如google；二、能够作缓存，加速访问资源；三、对客户端访问受权、上网进行认证；四、代理能够记录用户访问记录（上网行为管理），对外隐藏用户信息。

反向代理：实际运行方式是指以代理服务器来接受internet上的链接请求，而后将请求转发给内部网络上的服务器，并将从服务器上获得的结果返回给internet上请求链接的客户端，此时代理服务器对外就表现为一个服务器。

反向代理的用途：一、保证内网的安全，能够使用反向代理提供WAF功能，阻止web；二、大型网站，一般将反向代理做为公网访问地址，Web服务器是内网。
 
 ## 基于七层交换技术的负载均衡
 ## 负载均衡算法
 [Nginx org官方文档](https://nginx.org/en/docs/http/load_balancing.html)
 [Nginx Plus 官方文档](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
 
- **轮循**(Round Robin)：把来自用户的请求轮流分配给内部的服务器，从服务器1开始，直到服务器N，然后重新开始循环

-  **加权轮询**：不同的服务器配置，抗压能力，负载不同的时候，约定权重，根据权重进行轮询。
	
- **最少连接数**(Least Connection)：轮询法不能识别在给定的时间里保持了多少连接，因此可能发生，服务器B服务器收到的连接比服务器A少但是它已经超载，比如B服务器虽然收到的连接数少，但是都保持连接，没有断开，A服务器虽然收到的连接数多，但是实际大都断开连接，这时候就需要优先选择活跃连接数最少的服务器。

- **源IP哈希**(Source IP Hash)：根据客户端IP地址进行转发。保证某个特定的客户端最终会访问到同一个服务器主机，服务器需要保存用户信息的时候经常使用这种算法。

- **通用哈希**(Generci Hash):根据一个 自定义键值确定最终转发的目的地，该键值可以是字符串,变量或者组合如源 IP 和端口号

- **最短响应时间**(Least Time):NGINX Plus选择具有最低平均延迟和最低活动连接数的服务器

 - **随机**(Random):随机选择服务器(NGINX Plus)
 
## session和cookie

>首先讲HTTP协议是无状态的协议，但是服务端需要记录用户的状态，所以就产生了session和cookie，cookie相当于你拿着学生证让门卫进行确认，这样就能从学生证上确认学生身份了，学生证由学生本人保存，里面存放了学生的个人信息，同理cookie是客户端浏览器保存用户信息的地方。
>	session相当于你告诉门卫你的名字，去学校后台数据库查找有没有这个学生，是在服务器端保存用户信息的地方

## 实验配置
**1.轮询**
![在这里插入图片描述](https://img-blog.csdnimg.cn/7978fcfa659543beafd8020ca4ea141e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**负载均衡服务端配置**

```bash
cp nginx.conf.default  nginx.conf #覆盖默认配置
```

```bash
 http {
 	upstream myapp1 {
        server 192.168.2.218;
        server 192.168.2.188;
        server 192.168.2.178;
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


**后端配置**
如果后端都没有安装nginx，报502错误，负载均衡器正常，后端故障。

**健康检测**：nginx 被动健康检测来自特定服务器的响应，如果失败错误，nginx 会将此服务器标记为失败，并会在一段时间内尝试避免选择此服务器用于后续的入站请求。
都安装好nginx，负载均衡器将采用轮询方式，轮流访问目的IP。

**2.加权轮询**

```bash
upstream myapp1 {
    server 192.168.2.218 weight=3;
    server 192.168.2.188;
    server 192.168.2.178;
}
```

**3.最小连接数**

```bash
 upstream myapp1 {
        least_conn;
		server 192.168.2.218;
        server 192.168.2.188;
        server 192.168.2.178;
    }
```
**4.IP哈希**

```bash
upstream myapp1 {
       ip_hash;
	   server 192.168.2.218;
       server 192.168.2.188;
       server 192.168.2.178;
}
```
**5.Genric哈希**

```bash
upstream myapp1 {
       hash $request_uri consistent;
	   server 192.168.2.218;
       server 192.168.2.188;
       server 192.168.2.178;
}
```
**6.最短响应时间**
```bash
upstream myapp1 {
       least_time header;
	   server 192.168.2.218;
       server 192.168.2.188;
       server 192.168.2.178;
}
```
**7.随机Random**
```bash
upstream myapp1 {
       random;
	   server 192.168.2.218;
       server 192.168.2.188;
       server 192.168.2.178;
}
```

## Realip模块使用获取用户真实IP地址
**FULLNAT**：将返回的数据包源地址改为自己(SNAT)，目的地址改为客户端(DNAT)，源地址和目的地址都变。nginx负载均衡就是使用了FULLNAT模式

**问题**：Nginx反向代理无法获取到用户客户端的真实IP地址，因为FULLNAT模式已经将源地址和目的地址都修改了。

**解决方法**

1.X-REAL-IP
- .在负载均衡器上修改http请求报文字段，添加一个X-REAL-IP字段，X-Real-IP这个变量可以自定义，不区分大小写。

	```bash
	location / {
	           # root   html;
	           # index  index.html index.htm;
	           proxy_pass http://myapp1;
	           proxy_set_header   X-Real-IP        $remote_addr;
	 }
	```
- 在后端conf/nginx.conf添加这个字段`$http_x_real_ip`取消注释

	```bash
	log_format  main  '$remote_addr - $http_x_real_ip $remote_user [$time_local] "$request" '
	                  '$status $body_bytes_sent "$http_referer" '
	                  '"$http_user_agent" "$http_x_forwarded_for"';
	
	access_log  logs/access.log  main;
	```

	```bash
	nginx -s reload #刷新
	tail -f /logs/access.log #查看日志
	```
	真实地址为192.168.2.14
![在这里插入图片描述](https://img-blog.csdnimg.cn/300c9a62ac8349f6b5c5519b015bf61d.png)

2.使用realip模块
- 在编译安装前，使用 --with-http_realip_module 启用此模块
- 在后端conf/nginx.conf location添加一行`set_real_ip_from 负载均衡器IP地址;`

	```bash
	  location / {
	            root   html;
	            set_real_ip_from  192.168.2.5;
	            index  index.html index.htm;
	        }
	```
	
	```bash
	nginx -s reload #刷新
	tail -f /logs/access.log #查看日志
	```
	真实地址为192.168.2.14
	![在这里插入图片描述](https://img-blog.csdnimg.cn/5ef15e57f3274720981c6d58134905f6.png)
## 基于四层交换技术的负载均衡
## 四层、七层负载均衡区别
七层即应用层，就是基于 URL 等应用层信息的负载均衡。只支持http
四层即传输层，就是基于IP和端口的负载均衡。支持http,mysql,dns,ftp

## LVS和Nginx负载均衡区别
- 层数：LVS只工作在第四层传输层，支持http,mysql,dns,ftp，Nginx还可以工作在七层，针对域名、目录结构分流。
-  LVS效率高，LVS内核内置
- LVS配置性低，没有太多可配置的选项，所以工作稳定
- LVS缺乏持续的更新，Nginx被F5收购，有源源不断的支持。
	

## 实验配置
[官方文档](https://nginx.org/en/docs/stream/ngx_stream_core_module.html)

```bash
worker_processes auto;
error_log /usr/local/scwangyutao99/logs/error.log info;
events {
    worker_connections  1024;
}
stream {
    upstream backend {
       # hash $remote_addr consistent;
        server 192.168.2.178:80 weight=5;
        server 192.168.2.218:80 weight=5;
        server 192.168.2.188:80 weight=5;
    }
    upstream dns {
       server 192.168.0.1:53;
       server 192.168.0.3:53;
    }
    server {
        listen 80;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }
    server {
        listen 127.0.0.1:53 udp reuseport;
        proxy_timeout 20s;
        proxy_pass dns;
    }
}  
```
