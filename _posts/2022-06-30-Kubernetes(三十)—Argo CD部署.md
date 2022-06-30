---
layout: article
title: Kubernetes(三十)—Argo CD部署
tags: Kubernetes
category: blog
date: 2022-06-30 18:33:00 +08:00
mermaid: true
---
## 简介

**基于Kubernetes的声明式Gitops持续部署工具**

1.应用定义，配置和环境变量管理等等。都是声明基于云原生的。
2.所有声明清单都存储在代码仓库中，受版本管理。
3.应用发布和生命周期管理都是自动化的，可审计的。

## 工作原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/0521d15ce23444b9835f43fd479eefc7.png)

## 安装 Argo CD

```bash
# 创建命名空间
kubectl create namespace argocd 
# 部署 argo cd
wget https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -n argocd -f install.yaml
```

## 安装 Argo CD CLI
**下载Argo CD CLI**

Argo CD CLI 是用于管理 Argo CD 的命令行工具
```bash
wget https://github.com/argoproj/argo-cd/releases/download/v2.2.2/argocd-linux-amd64
chmod +x argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
```
**访问Argo CD API Server**

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc argocd-server -n argocd
kubectl get pod -n argocd -o wide

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2a8b48fe41a44f218f20542309fe2316.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/af868ac0958f420aa879f24768f22782.png)

**获取 Argo CD 密码**

默认情况下 admin 帐号的初始密码是自动生成的，会以明文的形式存储在 Argo CD 安装的命名空间中名为 argocd-initial-admin-secret 的 Secret 对象下的 password 字段下，我们可以用下面的命令来获取

```bash
kubectl -n argocd get secret \
argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```
**登录Argo CD**

```bash
argocd login 10.10.221.58

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2536c7df8e5e417e9c8760d5a29695f1.png)

对之前的密码进行修改

```bash
argocd account update-password
```
做一个端口转发

![在这里插入图片描述](https://img-blog.csdnimg.cn/2fc510b9af724feca4a9cf93f3e91231.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/731daf6c6482416a83deb790503b9c9a.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/09211ac0d5eb446eb2ce2b3fe32a9a5d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/cc344c4a2c7949269c9b010e32061f6a.png)

> 仓库地址: https://gitee.com/cnych/argocd-example-apps

![在这里插入图片描述](https://img-blog.csdnimg.cn/81c83dd030794bce8806b1dec5ef3a1d.png)

同步策略设置为Manual，所以需要手动点击SYNC进行同步


![在这里插入图片描述](https://img-blog.csdnimg.cn/aa73359ee61b453f85ace7ae3a1bebb6.png)

同步完成后可以看到我们的资源状态

![在这里插入图片描述](https://img-blog.csdnimg.cn/b75288f72cd246f9a70c39bc13b36a90.png)
