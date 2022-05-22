---
layout: article
title: Kubernetes(二十二)—Helm:K8s的应用包管理工具
tags: Kubernetes
category: blog
date: 2022-05-22 19:25:00 +08:00
mermaid: true
---
## 简介

Helm是一个 Kubernetes 应用的包管理工具，helm 类似于Linux系统下的包管理器，如yum/apt等，python中pip，node中的npm，可以方便快捷的将之前打包好的yaml文件快速部署进kubernetes内，方便管理维护。
## 使用helm的优势
之前部署一个应用的过程：
编写yaml文件 --> 创建service对外暴露 --> 通过ingress域名访问

**不使用helm方式的缺点**
只适合简单的应用。实际上我们部署微服务项目，可能有几十个服务，用这种方式就不合适了。需要维护大量的yaml文件。

**使用helm的优势**
- 可以将大量yaml文件作为一个整体进行管理
- 实现yaml文件的高效复用。
- 实现应用级别的版本管理

总之，helm的作用就是让我们在k8s中部署应用更高效。
## Helm的整体架构

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb9acdcd510d4f5cb81dc9c3946977d8.png)

**Helm主要包括以下组件**

- **Chart**：Chart是一个Helm的程序包，软件包，包含了运行一个K8S应用程序所需的镜像、依赖关系和资源定义等，也就是包含了一个应用所需资源对象的YAML文件，通常以.tgz压缩包形式提供，也可以是文件夹形式。

- **Repository（仓库）**：是Helm的软件仓库，Repository本质上是一个Web服务器，该服务器保存了一系列的Chart软件包供用户下载，并且提供了该Repository的Chart包的清单文件便于查询。

- **Config（配置数据）**：部署时设置到Chart中的配置数据。

- **Release**：基于Chart和Config部署到K8S集群中运行的一个实例。一个Chart可以被部署多次，每次的Release都不相同。

**Helm的工作流程**
- 开发人员将开发好的Chart上传到Chart仓库。
- 运维人员基于Chart的定义，设置必要的配置数据（Config），使用Helm命令行工具将应用一键部署到K8S集群中，以Release概念管理后续的更新回滚等。
- Chart仓库中的Chart可以用于共享和分发。

## Helm的版本
**V2版本**Helm依赖Tiller组件。Tiller组件用于接收Helm客户端发出的指令，与K8S的API Server交互，完成资源对象的部署和管理。
**V3版本**Helm不再使用Tiller组件，而是将API Server交互的功能整合到Helm客户端程序中。
## Helm安装
```bash
bash <(curl -s -L https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3)
helm version
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/c48bca878b1140de9d35d29be4e9b7fd.png)

**代理访问**
如果不走代理可能无法成功下载
把代理服务器地址写入shell配置文件.bashrc或者/etc/profile
```bash
export http_proxy="http://localhost:port"
export https_proxy="http://localhost:port"
```

```bash
source /etc/profile
```


**添加Chart仓库**
```bash
helm repo add stable http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts 
helm repo update
helm repo list
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f80ae2bf982f4d9794cbe425c92c1a53.png)

**移除Chart仓库**
```bash
helm repo remove stable
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/89672d3fd51c4c1a9bf66523937f64b4.png)

