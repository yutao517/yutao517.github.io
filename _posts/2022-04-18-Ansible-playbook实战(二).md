---
layout: article
title: Ansible-playbook实战（二）
tags: Ansible
category: blog
date: 2022-04-18 20:13:00 +08:00
mermaid: true
---
## 模版配置文件安装redis
上次我们使用了简单的方法copy配置文件安装redis，这次我们使用内置变量和模版来做这件事情。

```bash
cd /root/121.36.40.218/etc/
#进入配置文件放置的目录
cp redis.conf redis.conf.j2 
#复制配置文件为redis.conf.j2 
vim redis.conf.j2 
#编辑变量使其成为模版
:61 #进入第61行绑定IP地址的一行
bind {{ansible_facts['eth0']['ipv4']['address']}}
:84
port {{redis_port}}
#设置IP地址端口为变量
```

```bash
ansible all -m setup
#查看模版，确实有ansible_facts这个变量
```

```bash
vim /root/playbooks/redis_third.yaml
#写剧本3
```

```bash
- hosts: all
  remote_user: root
  vars:
    redis: /root/121.36.40.218/etc/redis.conf.j2
    redis_port: 6399
  tasks:
  - name: install redis
    yum: name=redis state=latest
  - name: start redis
    service: name=redis state=started
  - name: copy config
    template: src={{redis}} dest=/etc/redis.conf owner=redis
    #这里使用了模版
    notify: restart redis
    tags: configfile
  - name: start redis
    service: name=redis state=started
  handlers:
  - name: restart redis
    service: name=redis state=restarted                                                           
```

```bash
ansible-playbook redis_third.yaml 
#运行剧本
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a8d7bb67e94d4b2394adda3341f140fb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

这时候我们去远程主机看一下redis的绑定IP和端口

![在这里插入图片描述](https://img-blog.csdnimg.cn/ce09c8b44b334d72a811a1d4a03b1a4a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
可以看到绑定的是自己IP和我们写的端口6399
