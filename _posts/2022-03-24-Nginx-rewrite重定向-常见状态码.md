---
layout: article
title: Nginx-rewrite重定向-常见状态码
tags: Nginx
category: blog
date: 2022-03-24 000000 +0800
mermaid: true
---
>重定向可以做**防盗链**,防盗图片

```bash
	location ~* \.(gif|jpg|png|swf|flv)$ { 
    valid_referers none blocked www.yutao.co;  #有效的引用
    if ($invalid_referer) { 
    rewrite ^/ http://www.yutao.co/return.html; #重定向到这个网站
    #return 403;  或者直接返回403 
  				 } 
	}
```
## Nginx重定向的方式

**1.nginx rewrite**
```bash
 location / {
            rewrite ^/(.*) https://www.qq.com/$1 permanent;
            }
```

```bash
location / {
		   return 301 https://www.qq.com$request_uri;
		   }
```

rewrite最后一项flag参数

| 标记符号 | 说明|
|--|--|
|  last| 本条规则匹配完成后继续向下匹配新的location URL规则 |
|break|本条规则匹配完成后中止，不在匹配任何规则
|redirect|返回302临时重定向
|permanent|返回301永久重定向

**2.location proxy_pass功能**

```bash
location / {
		   proxy_pass https://www.qq.com;
		   }
```
3.**在首页添加跳转代码**

```html
<html>
        <head>
              <meta http-equiv="refresh" content="3;url=http://www.yutao.co"> #三秒后重定向到www.yutao.co
        </head>
        <body>
                <p>正在重定向... 
        </body>
<html>
```
   
4.**dns域名解析**





## 常见状态码

[状态码](https://zh.m.wikipedia.org/zh-hans/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)

1XX 它表示请求已经被接受，正在继续处理，这种响应是临时响应，不会返回响应体。

2XX 成功处理并返回，它表示在服务器内已经被接收，被知晓，并处理完成。

3XX 重定向功能，告知客户端须要继续执行操做才能够完成请求。

301： 请求的URL指向的资源已经被删除；但在响应报文中经过首部Location指明源如今所处的新位置；

302： 响应报文Location指明资源临时新位置；

304： 客户端发出了条件式请求，但服务器上的资源不曾发生改变，则经过响应此状态码通知客户端；

4XX 出现问题，和客户端有关系，好比401表示权限问题，404表示访问了一个不存在的URL。

401： 须要输入帐号和密码认证方能访问资源；

403禁止访问:一、将nginx.config的user改成和启动用户一致；二、配置文件中index index.html index.htm这行指定的文件。三、修改web目录的读写权限，或者是把nginx的启动用户改为目录的所属用户，四、关闭/etc/selinux/config；

404： 服务器没法找到客户端请求的资源；缘由：有可能location路径写错 了，要在nginx.conf/index.html 文件中添加缺失文件。

5XX 出现问题，和服务端有关，好比500表示内部错误，缘由是：ASP语法出错、ACCESS数据库链接语句出错、文件引用与包含路径出错(如未启用父路径)、使用了服务器不支持的组件，如FSO等。

502 Bad Gateway错误是FastCGI有问题， 1.FastCGI进程是否已经启动 ；2.FastCGI worker进程数是否不够； 3.增长缓冲区容量大小；

503： 服务不可用，临时服务器维护或过载，服务器没法处理请求；排查方法：一、管理员可能关闭应用程序池以执行维护。二、当请求到达时应用程序池队列已满。三、应用程序池的性能选项卡的请求队列限制所填的数值过小，默认为1000。

504 网关超时；1. 优化业务代码:一个接口调用超过一分钟，必定有能够优化的地方，看看数据库或者接口的调用是否合理，是否能够合并请求。2. 修改Nginx的服务器配置：若是实在是优化不了了，能够把Nginx的超时时间上调。


**200** ：服务器已成功处理了请求

**301** ：永久重定向

**302** ：临时重定向

**304** ：not modifiled 浏览器里缓冲的内容和nginx服务器里的内容一样，也就是自从上次请求后，请求的网页未修改过。

**403** ：Forbidden 服务器拒绝请求。

**404**： not found 服务器找不到请求的网页。

**500**：服务器内部错误

**502**：网关故障，负载均衡正常，后端服务器挂了

**503**： 服务器暂时不可达


