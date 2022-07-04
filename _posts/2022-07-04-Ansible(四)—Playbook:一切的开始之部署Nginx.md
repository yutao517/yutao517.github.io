---
layout: article
title: Ansible(四)—Playbook:一切的开始之部署Nginx
tags: Ansible
category: blog
date: 2022-07-04 11:23:00 +08:00
mermaid: true
---
## 一个简单的Playbook

```bash
mkdir -p playbook/ansible-nginx
cd playbook/ansible-nginx
mkdir files templates
vim web-notls.yml
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/97ab1bae153e4a32affae6c61bb9f40b.png)

```bash
- name: Configure webserver with nginx
  hosts: master
  become: True
  tasks:
  - name: Nginx install nginx
    yum:
      name: nginx
      state: latest

  - name: copy nginx config file
    copy:
      src: files/nginx.conf
      dest: /etc/nginx/nginx.conf

  - name: copy index.html
    template:
      src: templates/index.html.j2
      dest: /usr/share/nginx/html/index.html
      mode: 0644

  - name: restart nginx
    service: name=nginx state=restarted
```

## 定义Nginx的配置文件

>files/nginx.conf
```bash
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;
        error_page 404 /404.html;
        location = /404.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
```

## 定义一个定制的首页

>templates/index.html.j2

```bash
<html>
    <head>
        <title>Welcome to ansible</title>
    </head>
    <body>
        <h1>nginx,configured by Ansible</h1>
        <p>If you can see this,Ansible successfully installed nginx.</p>
        <p>{{ ansible_managed }}</p>
    </body>
</html>
```

## 运行这个Playbook

```bash
ansible-playbook web-notls.yml
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/dbf5c7ec06bc42208bed318fc18492b1.png)

## 验证效果
看到自定义的HTML页面是，输出结果如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/485b8661b7f44ca8afd5289d8f15b1ea.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/497a753a6a2b4d8095fe456ee9992f81.png)
