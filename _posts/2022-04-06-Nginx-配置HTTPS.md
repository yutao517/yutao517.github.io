---
layout: article
title: Nginx-配置HTTPS
tags: Nginx
category: blog
date: 2022-04-06 000000 +0800
mermaid: true
---

[HTTP工作原理](https://cloud.tencent.com/developer/news/732538)

## HTTPS工作流程
[对称加密与非对称加密的区别](https://www.html.cn/qa/other/20015.html)
- 对称加密: 加密和解密的秘钥使用的是同一个。

- 非对称加密: 与对称加密算法不同，非对称加密算法需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）。

[HTTPS深入原理](https://juejin.cn/post/6844903830916694030)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/5bf07546-198d-48ac-80e7-6b11c5ff766c)

- Client发起一个HTTPS的请求，根据RFC2818的规定，Client知道需要连接Server的443（默认）端口。
- Server把事先配置好的公钥证书（public key certificate）返回给客户端。
- Client验证公钥证书：比如是否在有效期内，证书的用途是不是匹配Client请求的站点，是不是在CRL吊销列表里面，它的上一级证书是否有效，这是一个递归的过程，直到验证到根证书（操作系统内置的Root证书或者Client内置的Root证书）。如果验证通过则继续，不通过则显示警告信息。
- Client使用伪随机数生成器生成加密所使用的对称密钥，然后用证书的公钥加密这个对称密钥，发给Server。
- Server使用自己的私钥（private key）解密这个消息，得到对称密钥。至此，Client和Server双方都持有了相同的对称密钥。
- Server使用对称密钥加密“明文内容A”，发送给Client。
- Client使用对称密钥解密响应的密文，得到“明文内容A”。
- Client再次发起HTTPS的请求，使用对称密钥加密请求的“明文内容B”，然后Server使用对称密钥解密密文，得到“明文内容B”。


**HTTP1.1与1.0的区别、HTTP与HTTPS的区别？**

- 引入持久链接，即 在同一个TCP的链接中可传送多个HTTP请求 & 响应
- 多个请求 & 响应可同时进行、可重叠；
- 引入更加多的请求头 & 响应头

![image](https://user-images.githubusercontent.com/62100249/197145222-9c85c97f-ea90-42d6-ba58-24c3a12f7f61.png)


## 配置https环境

**阿里云申请免费的证书**

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/186fa457-24cd-4498-ac1f-e4e4d2e9bfc5)

立即购买免费证书

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/df495547-8603-400f-9eda-b4ca2c5a7b81)

创建证书，证书申请

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/11b49b9c-0807-449f-95a3-58c66a0ddcb3)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/ef15a737-e866-46a4-975a-7d1dc3780594)




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

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/831e74f5-43e1-4f24-a4ed-db397251400e)

