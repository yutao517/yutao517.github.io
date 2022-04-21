---
layout: article
title: Docker(十)—Compose容器启动WordPress
tags: Docker
category: blog
date: 2022-04-21 21:28:00 +08:00
mermaid: true
---
**创建一个空的项目目录**
```bash
mkdir my_wordpress/
cd my_wordpress/
```
**创建docker-compose.yml文件**
用于启动 WordPress博客的文件和一个带有卷挂载的单独MySQL实例以实现数据持久性

```bash
vim docker-compose.yml
```

```bash
version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      #填写自己的用户名密码
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```

```bash
docker-compose up -d
#后台启动
```

浏览器访问网站，个性化设置

![在这里插入图片描述](https://img-blog.csdnimg.cn/bbcc7a9ffa0649708b666dc68feef80b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

```bash
docker-compose down --volumes
#删除所有数据
```
网站下线
