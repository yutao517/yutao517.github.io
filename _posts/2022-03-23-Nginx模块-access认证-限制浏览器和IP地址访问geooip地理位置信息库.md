---
layout: article
title: Nginx模块-access认证-限制浏览器和IP地址访问geooip地理位置信息库
tags: Nginx
category: blog
date: 2022-03-23 000000 +0800
mermaid: true
---

## Nginx常用模块

core模块；access访问控制模块；auth_basic基本认证模块；autoindex索引模块；log日志模块；gzip压缩模块；stub_status状态模块；geoip模块；rewrite重定向模块； proxy模块；realip模块；stream模块；upstream模块；
这些模块博客都会用到

**nginx中rewrite有哪几个flag标志位（last、break、redirect、permanent），说一下都什么意思？经常使用的Nginx模块，用来作什么的？在proxy模块中你配置过哪些参数？**

**flag标志位**
- last : 至关于Apache的[L]标记，表示完成当前的rewrite规则
- break : 中止执行当前虚拟主机的后续rewrite指令集
- redirect : 返回302临时重定向，地址栏会显示跳转后的地址
- permanent : 返回301永久重定向，地址栏会显示跳转后的地址、301和302不能简单的只返回状态码，还必须有重定向的URL，这就是return指令没法返301,302的缘由了。

last 和 break 区别有点难以理解：

last通常写在server和if中，而break通常使用在location中，last不终止重写后的url匹配，即新的url会再从server走一遍匹配流程，而break终止重写后匹配，break和last都能组织继续执行后面的rewrite指令。

**Nginx模块**

- rewrite模块，实现重写功能 
- access模块：来源控制 
- ssl模块：安全加密 
- ngx_http_gzip_module：网络传输压缩模块 
- ngx_http_proxy_module 模块实现代理 
- ngx_http_upstream_module模块实现定义后端服务器列表 
- ngx_cache_purge实现缓存清除功能

**proxy模块**

配置过:proxy_set_header，proxy_connect_timeout，proxy_send_timeout、proxybuffer。

## access限制IP地址访问

```bash
  error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
      }

        location = /wyt_status{
        stub_status;
        deny 192.168.2.3;
        allow 192.168.2.14;
        }
```
## auth_basic认证

htpasswd文件存放路径--->conf目录
htpasswd命令生成htpasswd文件
```bash
		yum provides htpasswd
		yum install httpd-tools -y
		htpasswd -c htpasswd wyt #在conf目录下生成用户名密码文件

```
配置文件添加location
```bash
         location  /mp3 {
            root   html/wyt;
         auth_basic "wyt";
         auth_basic_user_file htpasswd;
         autoindex on;
         }
```
    
![在这里插入图片描述](https://img-blog.csdnimg.cn/75bfa2e0bdb9462f93887ed11baee1a4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

** 限制浏览器访问**
在HTTP协议请求报文有User-Agent记录了浏览器信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/e568b03c74974cde938c7b779a81ff28.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_17,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9195962cc7cc440196ff627bf08714cd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

## 对浏览器进行访问

~区分大小写
~*不区分大小写

```
if ($http_user_agent !~* chrome)
{
return 403;
}
```
## 对IP地址进行访问*
```
if ($remote_addr ~* 192.168.2.14)
{
return 403;
}
```
火狐浏览器访问出错403

![在这里插入图片描述](https://img-blog.csdnimg.cn/27ac50fb7d6f4957bf5c00826100e353.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

## geoip地理信息ip

编译前打开geoip库

下载geoip地理信息ip库

http添加

```bash
geiop_city  /usr/local/scwangyutao/conf/GeoLiteCity.dat;
```

sever添加
```bash
 if ($geoip_city != 'changsha' ){
	return 403;
	}
```


