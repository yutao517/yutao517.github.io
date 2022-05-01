---
layout: article
title: Nginx-配置HTTPS
tags: Nginx
category: blog
date: 2022-04-06 000000 +0800
mermaid: true
---

## 配置https环境

**阿里云申请免费的证书**

找到SSL证书![在这里插入图片描述](https://img-blog.csdnimg.cn/4897bcac60064858a632b0a44c81aaa8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
立即购买免费证书
![在这里插入图片描述](https://img-blog.csdnimg.cn/8df9ef898c4a4651b4a187acfa925e84.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
创建证书，证书申请
![在这里插入图片描述](https://img-blog.csdnimg.cn/3148dc83e7514f80b30d5fd35932cab2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)


![在这里插入图片描述](https://img-blog.csdnimg.cn/1a15328679d442cdaec349e57c352c1e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)



**服务器配置**

环境：nginx的安装目录是/usr/local/scwangyutao99
二进制源包在/scwangyutao99/nginx-1.21.4目录下


```bash
cd /wangyutao99/nginx-1.21.4/ #切换到源码包

./configure --prefix=/usr/local/scwangyutao99 --with-http_stub_status_module --with-http_ssl_module #编译https配置文件

make #编译

cp /usr/local/scwangyutao99/sbin/nginx /usr/local/scwangyutao99/sbin/nginx.bak #备份原来的nginx

nginx -s stop #停止nginx

cp ./objs/nginx /usr/local/scwangyutao99/sbin/ #将编译好的nginx覆盖掉原有的nginx,提示输入yes

nginx #启动

```




**配置nginx**

将下载下来的ssl证书解压放置到/usr/local/scwangyutao99/conf/
```bash
server {
        listen       443 ssl;
        server_name  www.yutao.co;
        ssl_certificate      7558164_yutao.co.pem;  #填写解压的pem文件
        ssl_certificate_key  7558164_yutao.co.key;  #填写解压的key文件

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }
    
 server {
        listen       80;
        server_name  www.yutao.co;

        location / {
        rewrite ^(.*) https://$server_name$1 permanent;
                   }       
         }
#http重定向
```
访问网站已经是安全链接
![在这里插入图片描述](https://img-blog.csdnimg.cn/88defd71ee22473bae5a6df7801de578.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_15,color_FFFFFF,t_70,g_se,x_16)
