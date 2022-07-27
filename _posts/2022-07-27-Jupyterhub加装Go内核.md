---
layout: article
title: Jupyterhub加装Go内核
tags: Jupuyerhub
category: blog
date: 2022-07-27 17:05:00 +08:00
mermaid: true
---
## 安装GO

```bash
wget https://studygolang.com/dl/golang/go1.17.6.linux-amd64.tar.gz
tar -zxvf go1.17.6.linux-amd64.tar.gz
useradd admin
mv go /usr/local
```

**配置环境变量**

```bash
echo 'export PATH=$PATH:/usr/local/go/bin'>>/etc/profile
echo 'export GOPATH=/home/go/go'>>/etc/profile
```

```bash
go version
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/816afc05e2e9461992d83d5863f5c59a.png)

**配置Go代理**
```bash
export GO111MODULE=on
export GOPROXY=https://goproxy.cn
```
 **查看jupyte内核**

```bash
jupyter kernelspec list
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9f30dbab6cf402e87bd57f5c4936e4b.png)

## gophernotes - 在 Jupyter 笔记本和交互中使用 Go
[https://github.com/gopherdata/gophernotes](https://github.com/gopherdata/gophernotes)

```bash
 go install github.com/gopherdata/gophernotes@v0.7.5
 mkdir -p /usr/local/share/jupyter/kernels/gophernotes
 cd /usr/local/share/jupyter/kernels/gophernotes
 cp "$(go env GOPATH)"/pkg/mod/github.com/gopherdata/gophernotes@v0.7.5/kernel/*  "."
 chmod +w ./kernel.json # in case copied kernel.json has no write permission
 sed "s|gophernotes|$(go env GOPATH)/bin/gophernotes|" < kernel.json.in > kernel.json
```
要确认gophernotes二进制文件已安装在 GOPATH 中，请直接执行：

```bash
 "$(go env GOPATH)"/bin/gophernotes
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f784ac78356b44de84fca79813fde219.png)

## 打开Jupyterhub
admin用户创建密码可以登录使用
登录admin用户

![在这里插入图片描述](https://img-blog.csdnimg.cn/2ae3cb5ab8de4de0aba059a3ecb09523.png)

登录其他用户将存在权限问题，这是因为内核的目录在admin用户，其他用户无权限。![](https://img-blog.csdnimg.cn/f6a43ec86f1f42c6aab4bea44c7884d0.png)

