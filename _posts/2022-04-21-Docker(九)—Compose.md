---
layout: article
title: Docker(九)—Compose
tags: Docker
category: blog
date: 2022-04-21 20:18:00 +08:00
mermaid: true
---
## 概述
- Compose是Docker容器进行编排的工具，定义和运行多容器的应用，可以一条命令启动多个容器，使用Docker Compose不需要使用shell脚本来启动容器。
- Compose项目来源于之前的fig项目，使用Python语言编写，与docker/swarm配合度很高。
- Compose是用于定义和运行多容器Docker应用程序的工具，通过Compose，可以使用yaml文件来配置应用程序的服务。然后使用一个命令就可以从配置中创建并启动所有服务。

## 安装 

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
#如果比较慢就用下面的链接
```

```bash
sudo curl -L "https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose -v
```
## 入门
创建目录
```bash
mkdir /root/composetest
cd /root/composetest
vim app.py
```
创建一个名为的文件app.py
```bash
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```
创建另一个名为requirements.txt的文件
```bash
vim requirements.txt
>flask
>redis
```
**创建 Dockerfile**

```bash
vim Dockerfile
```

```bash
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```
**在 Compose 文件中定义服务**
创建一个名为docker-compose.yml的文件

```bash
vim docker-compose.yml
```

```bash
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
    ports:
      - "6379:6379"
```

```bash
docker-compose up -d
#后台启动
```

因为网络环境问题，我直接在AWS上运行，可以看到效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/e539c2a4043b463e9bdc75d760cf39b8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
