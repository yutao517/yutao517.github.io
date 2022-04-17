---
layout: article
title: Ansible-playbook实战（一）
tags: Ansible
category: blog
date: 2022-04-17 22:13:00 +08:00
mermaid: true
---
## 简介
playbook是Ansible用于配置，部署，和管理被控节点的剧本。

## playbook的核心元素

- hosts : playbook配置文件作用的主机
- tasks: 任务列表
- variables: 变量
- templates:包含模板语法的文本文件
- handlers :由特定条件触发的任务
- roles :用于层次性、结构化地组织playbook。roles能够根据层次型结构自动装载变量文件、tasks以及handlers等

![在这里插入图片描述](https://img-blog.csdnimg.cn/d7cf2dd84e3742cd98c011e215737169.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)


## 通过playbook安装redis服务
```bash
mkdir /root/playbooks
cd playbooks
vim redis_first.yaml
```

```bash
- hosts: all
  remote_user: root
  tasks:
  - name: install redis
    yum: name=redis state=latest
  - name: start redis
    service: name=redis state=started
```

```bash
ansible-playbook --syntax-check redis.yaml
#语法检测
```
没有报错，说明语法没有问题

![在这里插入图片描述](https://img-blog.csdnimg.cn/d7edf9f86b634498b5b6187e90b17902.png)

```bash
 ansible-playbook --list-hosts redis_first.yaml
 #列出运行任务的远程主机
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/8089f7e337fb4f238acc7695619021d1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

可以看到我这里只有一台121.36.40.218

```bash
ansible-playbook redis_first.yaml
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b9e26faef32b4431981ef0b8e224e377.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

上面的操作是直接安装redis服务并启动，并没有配置文件，我们在安装时提供配置文件。

## 配置文件安装redis
最简单的思路是先把配置文件fetch过来，然后修改配置文件再copy过去。

```bash
ansible 121.36.40.218 -m fetch -a 'src=/etc/redis.conf dest=/root'
#将121.36.40.218的redis配置文件fetch到assible主机
vim /root/121.36.40.218/etc/redis.conf
# 编辑配置文件修改bind 0.0.0.0
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f67c7ad3fbd24289b58cb5b57f040090.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

```bash
vim redis_second.yaml 
```

```bash
- hosts: all
  remote_user: root
  tasks:
  - name: install redis
    yum: name=redis state=latest
  - name: start redis
    service: name=redis state=started
  - name: copy config
    copy: src=/root/121.36.40.218/etc/redis.conf dest=/etc/redis.conf owner=redis
    notify: restart redis  #copy过去触发的动作
    tags: configfile       #任务标记名configfile
  - name: start redis
    service: name=redis state=started   
  handlers:                #和notify搭配，表示copy过去触发重启redis服务的动作      
  - name: restart redis
    service: name=redis state=restarted           
```
检验语法正确

![在这里插入图片描述](https://img-blog.csdnimg.cn/808a47d1c64345169b0666a7eb497803.png)

跑剧本
```bash
ansible-playbook redis_second.yaml
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cbca588c24be4b989135666855abf934.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

可以看到只是完成了两个动作，一个copy，一个重启，其他的都是已执行。但是我们如果想要只执行未执行过的动作可以通过标签
就是这一行：

```bash
tags: configfile       #任务标记名configfile
```
执行命令

```bash
ansible-playbook -t configfile redis_second.yaml
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/533f7d64b83f4b8e9819a7ce8d596a88.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

可以看到并没有执行安装，只执行了copy和重启的触发动作。
去远程主机查看redis已启动，配置文件已更新。

![在这里插入图片描述](https://img-blog.csdnimg.cn/1ff5ed41af5345b48203e1f3519e222f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c6e582f1e0bb4b8fa1581e307af509b8.png)
