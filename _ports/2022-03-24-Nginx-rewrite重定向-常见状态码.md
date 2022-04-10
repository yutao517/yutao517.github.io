
---
layout article
title Nginx-rewrite重定向-常见状态码
tags Nginx
category blog
date 2022-03-24 000000 +0800
mermaid true
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

**304** ：not modifiled 浏览器里缓冲的内容和nginx服务器里的内容一样，也就是自从上次请求后，请求的网页未修改过。
***
**404**： not found 服务器找不到请求的网页。
***
**403** ：Forbidden 服务器拒绝请求。
***
**200** ：服务器已成功处理了请求
***
**302** ：临时重定向
***
**301** ：永久重定向
***
**503**： 服务器暂时不可达
***
**502**：网关故障，负载均衡正常，后端服务器挂了
***
**500**：服务器内部错误
***

