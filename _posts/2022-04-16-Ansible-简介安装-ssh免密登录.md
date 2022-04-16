---
layout: article
title: Ansible-简介安装+免密通道
tags: Ansible
category: blog
date: 2022-04-16 21:19:00 +08:00
mermaid: true
---
## 简介
**Ansible**是一个基于Python开发的自动化运维工具，实现批量系统配置、批量程序部署、批量运行命令，本身没有批量部署的能力，真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架。ansible基于ssh来和远程主机通讯，不需要在远程主机上安装client/agents。

![架构图](https://img-blog.csdnimg.cn/903ec28be2d24e6b956473cae7a3488b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

## 模块
- Ansible：Ansible核心程序。
- HostInventory(主机清单)：记录由Ansible管理的主机信息，包括端口、密码、ip等，可以去操作那些主机。
- Playbooks：“剧本”YAML格式文件，多个任务定义在一个文件中，定义主机需要调用哪些模块来完成的功能，也就是告诉主机清单里的电脑做什么事情(精华)。
- ConnectionPlugins：连接插件，使Ansible和Host通信。
- CoreModules：核心模块，主要操作是通过调用核心模块来完成管理任务。
- CustomModules：自定义模块，完成核心模块无法完成的功能，支持多种语言。

## 安装
**环境**：ansible主机（centos7.9），远程主机（centos7.9 IP:121.36.40.218）

**建立免密通道**
```bash
ssh-keygen
#ssh-keygen直接回车命令默认在 ~/.ssh⽬录中⽣成SSH RSA公钥和私钥⽂件
cd /root/.ssh
ssh-copy-id -p22 -i id_rsa.pub root@121.36.40.218
#yes输入密码
#把本地的ssh公钥文件安装到远程主机的/root/.ssh下
ssh root@121.36.40.218 #免密登录成功
exit #退出
yum install epel-release -y
yum install ansible -y
cd /etc/ansible/
vim hosts 
#修改配置文件，添加主机清单
```

> [webservers]
121.36.40.218


如果没有开启免密登录也可以使用

```bash
[webservers]
121.36.40.218 ansible_ssh_user=''  ansible_ssh_pass=''
#输入自己的用户名密码
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/b67575c384d5437f87801b5a8a2f5254.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**测试效果**

调用copy模块传输一个文件

```bash
ansible all -m copy -a 'src=/root/test/scan.sh dest=/root'
```
效果同

```bash
scp /root/test/scan.sh root@121.36.40.218:~/
```
因为已经scp传输了该文件，所以是绿色字体可以理解为覆盖，黄色字体是改变，红色字体说明错误。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d91b024509ef4a3796e5af373354da73.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

```bash
ansible all -m script -a '~/weixin.sh wangyutao' 
#在我的ansible主机目录下刚好有一个脚本，用assible让所有远程主机运行一下。
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/40c1e53b89224638b5ed25f007d2ee07.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
