---
layout: article
title: Nginx的HTTPS和HTTP正向代理
tags: Nginx
category: blog
date: 2022-08-18 000000 +0800
mermaid: true
---
**HTTPS代理需要模块ngx_http_proxy_connect_module**

```bash
wget http://nginx.org/download/nginx-1.18.0.tar.gz
tar -xzvf nginx-1.18.0.tar.gz
yum install git -y
cd /home
git clone https://gitee.com/web_design_of_web_frontend/ngx_http_proxy_connect_module.git
yum -y install pcre-devel openssl openssl-devel
cd /root/nginxx-1.18.0
patch -p1 < /home/ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_1018.patch
./configure --add-module=/home/ngx_http_proxy_connect_module
./configure --prefix=/usr/local/nginx --with-http_ssl_module --add-module=/home/ngx_http_proxy_connect_module
make && make install
echo  "PATH=$PATH:/usr/local/nginx/sbin" >>/root/.bashrc
source /root/.bashrc
```
**配置文件修改**
```bash
    server {
        resolver 8.8.8.8; #指定DNS服务器IP地址，就用 8.8.8.8
        listen 8080;
        location / {
            proxy_pass http://$http_host$request_uri; #设定代理服务器的协议和地址
        }
    }
 
    server {
        resolver 8.8.8.8; #指定DNS服务器IP地址，就用 8.8.8.8 
        listen 8084;
        proxy_connect;
        proxy_connect_allow all;
        location / {
            proxy_pass https://$host$request_uri; #设定代理服务器的协议和地址
            proxy_set_header Host $host;
        }
    }
```

```bash
export https_proxy="https://152.32.167.156:8084"
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/84a6426568524aca840e9eed2e8e180d.png)
