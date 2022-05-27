---
layout: article
title: Linux—HTTP代理
tags: Linux 
category: blog
date: 2022-05-27 20:30:00 +08:00
mermaid: true
---
配置代理科学上网，访问Github
```bash
vim /etc/profile
```

```bash
export http_proxy=http://192.168.2.23:8080
export https_proxy=http://192.168.2.23:8080
```
输入自己的代理IP和端口，代理成功

```bash
source /etc/profile
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b639b9f9a52e4ca1b45cb9b88fac8cf0.png)
