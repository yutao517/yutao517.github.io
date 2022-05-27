---
layout: article
title: Kubernetes(二十六)—使用服务器搭建Harbor私有镜像仓库
tags: Kubernetes Project
category: blog
date: 2022-05-27 20:39:00 +08:00
mermaid: true
---
## 简介

>  Habor是由VMWare公司开源的容器镜像仓库。事实上，Habor是在Docker
> Registry上进行了相应的企业级扩展，从而获得了更加广泛的应用，这些新的企业级特性包括：管理用户界面，基于角色的访问控制，AD/LDAP集成以及审计日志等，足以满足基本企业需求。

 

# 安装

## 下载离线安装包

```bash
 wget https://github.com/goharbor/harbor/releases/download/v2.0.1/harbor-offline-installer-v2.0.1.tgz
 tar -zxvf harbor-offline-installer-v2.0.1.tgz
 cd harbor
```


## 配置对Harbor的HTTPS访问

域名解析到服务器，服务器从阿里云申请证书下载密钥之后将密钥上传到服务器/root/harbor/(申请方法自行百度)
[参考](https://blog.yutao.co/blog/2022/04/06/Nginx-%E9%85%8D%E7%BD%AEHTTPS.html)

```bash
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
```

```bash
hostname: harbor.yutao.co
http:
  port: 8080
https:
  port: 443
  certificate: /root/harbor/harbor.yutao.co.pem
  private_key: /root/harbor/harbor.yutao.co.key
harbor_admin_password: Harbor12345
database:
  password: root123
  max_idle_conns: 50
  max_open_conns: 100
data_volume: /data
clair:
  updaters_interval: 12
trivy:
  ignore_unfixed: false
  skip_update: false
  insecure: false
jobservice:
  max_job_workers: 10
notification:
  webhook_job_max_retry: 10
chart:
  absolute_url: disabled
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor
_version: 2.0.0
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - clair
    - trivy
```

## 部署Harbor
安装docker compose

```bash
sudo curl -L "https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose -v
```
通过使用prepare脚本为nginx配置启用HTTPS
通过以下compose启动Harbor：
```bash
./prepare
docker-compose up -d
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/cde06e1a21e94844b19325165194a7e8.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/e3c922b89caf40ac8119f172352fb9d1.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/0e784bc69ec7406dbe6891ec8d3b8245.png)

## 上传镜像

```bash
docker login harbor.yutao.co
Username: admin
Password: 
```

![](https://img-blog.csdnimg.cn/6abc8ef2ce1b4bf390fcf6a3565d6fac.png)

将自己阿里云私有镜像仓库的镜像上传到Harbor私有镜像仓库

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/yutao517/hello-app:1.0
docker tag registry.cn-hangzhou.aliyuncs.com/yutao517/hello-app:1.0 harbor.yutao.co/library/hello-app:1.0
docker push harbor.yutao.co/library/hello-app:1.0
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/fd1b211f49e54ceeb18f7a42e73a8cfb.png)

可以看到library已经有1个hello-app镜像

![在这里插入图片描述](https://img-blog.csdnimg.cn/7687dd88a8e842c0aed7f6541ab3fb69.png)

> 注意：如果使用HTTP而不是HTTPS连接Harbor镜像仓库，由于Harbor默认不允许HTTP连接，则必须在Docker客户端主机中使用--insecure-registry将Harbor镜像仓库访问地址添加进允许连接访问的列表中。即在Docker客户端主机中使用vi编辑器修改/etc/docker/daemon.json文件配置

```bash
vim /etc/docker/daemon.json
```
新增
```bash
{
"insecure-registries" : ["这里填自己的域名或ip:这里填HTTP暴露的端口（80可省略）", "0.0.0.0"]
}
```
## 拉取镜像

```bash
docker pull harbor.yutao.co/library/hello-app:1.0
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0b2f09a41df94901816fd1431cc562fb.png)

因为library在Harbor中是公开项目，所以不用登陆也能拉取镜像，但推送至library必须先登录具有library管理权限的用户。
