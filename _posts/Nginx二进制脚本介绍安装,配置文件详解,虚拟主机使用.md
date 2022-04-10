

## 简介

**Nginx**（发音同“engine X”）是异步框架的网页服务器，也可以用作反向代理、负载平衡器和HTTP缓存(第七层负载均衡和第四层负载均衡)

## 二进制脚本安装

```bash
bash <(curl -s -L https://cdn.jsdelivr.net/gh/yutao517/blog@main/bash/one-key-nginx-install.sh)
```

**nginx启动与关闭等常用命令**

```bash
#启动
echo "/usr/local/scwangyutao99/sbin/nginx" >>/etc/rc.local #写入开机自启动配置文件
chmod +x /etc/rc.d/rc.local #给可执行权限
#关闭
nginx -s stop #立即强制关闭，不管有没有正在处理的请求
nginx -s quit #优雅关闭，先处理完正在运行的请求
nginx -s reload #修改了nginx的配置文件，相当于刷新服务，启用新配置，不会中断业务
nginx -t #配置文件语法测试
cd /usr/local/scwangyutao99/logs #日志文件
#access.log记录正常的访问
#error.log记录出错的信息
#nginx.pid记录master的pid号

#如果是yum安装html一般在 /usr/share/nginx/html/,可执行文件在/usr/sbin/nginx,log文件在/var/log,配置文件在/etc/nginx
```

## nginx的master和woker的关系

![在这里插入图片描述](https://img-blog.csdnimg.cn/93a85ffb73d1421db4ae66d5cb355d5a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
master进程主要用来管理worker进程，包含：接收来自外界的信号，向各worker进程发送信号，监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程。
**注意:不是先到master然后调度woker负载均衡**

## nginx.conf 配置文件参数介绍

**nginx.conf作用：给nginx进程提供参数**
修改配置文件后一定要刷新服务因为**本质上是让Linux进程重新去加载配置文件，按照修改了的配置文件里的内容工作**

```bash
worker_processes  2;#进程设置值和CPU核心数一致
events {
    worker_connections  2048;#每一个工作进程的最大连接数量（并发数）， 最大的并发数，理论上，但是和cpu有关，做压力测试可知道实际。 理论上最大连接数 = worker_processes * worker_connections
}
http {
    include       mime.types; #区分资源的媒体类型
    default_type  application/octet-stream;
    日志格式log_format  main  '$remote_addr - $remote_user [$time_local] "$request"请求网址 '
                      '$status $body_bytes_sent 
      					响应报文状态发动的字节
          				"$http_referer"'
          				#从哪个网站引流跳转过来
                    	'"$http_user_agent" 浏览器
                    	"$http_x_forwarded_for"'代理转发;
    sendfile        on;  提升性能
    keepalive_timeout  65; #长连接，保持65秒的长连接
    server {
	listen  80;
	server_name www.wyt.com;
        access_log  logs/wyt.access.log  main;
        #日志会记录到wyt.access.log
        location / {
            root   html/wyt;
            index  index.html index.htm;#首页
        }
        error_page  404     #找不到的错误页面         /404.html;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html { #服务器内部错误
            root   html;
        }
		 
    }

}
```
## 文件描述符的问题
永久修改文件描述符的限制
```
vim /etc/security/limits.conf
#添加两行
*soft nofile 100000
*hard nofile 100000

```
## 完成三个域名的虚拟主机配置

```bash
vim /usr/local/scwangyutao99/conf/nginx.conf
```

  server_name www.wyt.com;

 

```bash
access_log  logs/wyt.access.log  main;

location / {
    root   html/wyt;     #需要去html文件下新建wyt目录下新建html页面
    index  index.html index.htm;
}

error_page  404              /404.html;
```

```bash
mkdir /usr/local/scwangyutao99/html/wyt
vim index.html
	#写入
	wyt
	wangyutao666
```

- 测试语法 `nginx -t`
- 刷新nginx服务  `nginx -s reload`
- 去另一台局域网主机修改hosts文件域名

```bash
echo "192.168.2.9 www.wyt.com" >> /etc/hosts
```

- 域名访问

```bash
curl www.wyt.com
```

- 可以看到新建的html网页
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/6e232309e3404a26a86a82d30cd9c838.png)
  同理三个域名
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/96df8ec2b7df443486a7467d01833be8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_18,color_FFFFFF,t_70,g_se,x_16)
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/66ac2744394d4323a194bfcb8239dc21.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

- 基于IP的虚拟主机：一个网站对应一个公网IP
- 基于端口的虚拟主机：一个网站一个端口
- 基于域名的虚拟主机：通过不同的域名区别开不同的网站，但这些网站公用一个公网ip和使用相同的端口80
  优点：节省服务器
    			缺点：一台虚拟服务器受到攻击，其他网站会受到牵连，共用CPU,内存，磁盘，带宽。一台服务器的访问量特别大，会导致其他网站异常。

  			



 
