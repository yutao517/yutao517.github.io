---
layout: article
title: 阿里云云原生助理工程师ACA认证(六)—Serverless理念函数计算FC
tags: 阿里云云原生容器工程师 Kubernetes Docker
category: blog
date: 2022-09-26 15:10:00 +08:00
mermaid: true
---
## 云计算平台的优势和局限性
2009年加利福尼亚大学伯克利分校的论文:Above the Clouds: A Berkeley View of CloudComputing对云计算总结了6点优势:
- 无限可用的计算资源（理论上)
- 用户再也不需要承担服务器运维的工作和责任
- 服务的按需付费成为可能
- 超大型数据中心的使用成本显著降低
- 通过可视化资源管理，运维操作的难度大大降低
- 得益于分时复用，物理硬件的利用率大大提高


相比较传统模式，云计算模式着重解决了数据中心商业模式的转变和服务器的虚拟化。开发人员使用服务器的方式并没有太多改变

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/15bfc485-a2c8-4dba-94a9-676c84b9496c)

云计算的优势:
- 用户按服务器时长付费
- 降低了硬件的组建，维护成本，系统迁移成本较低。

云计算的局限性:
- 用户无法做到按使用量付费
- 服务器仍然需要用户进行维护
- 应用部署仍然不够便捷

## 需求催生新的架构升级
2015年亚马逊推出了AWS Lambda产品。提出了Cloud Function和Serverless的概念。(Serverless并不是说真的没有服务器了，只是开发人员不需要再关注服务器了）引起了业界的广泛关注。

![image](https://user-images.githubusercontent.com/62100249/192214287-eeac8d72-4e5a-4a57-828d-515cf3ceb639.png)


Serverless平台包装了云计算平台下应用所需要的全部资源。是开发者不在需要关注laas层的资源占用情况。


## Serverless的技术优势

2019年加利福尼亚大学伯克利分校再次发表论文: Cloud Programming Simplified: A BerkeleyView on Serverless Computing。

**对Serverless总结了几点特点**

- 弱化了储存和计算之间的联系
- 代码的执行不再需要手动分配资源
- 按计算量精确计费

## Serverless带来的技术升级

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/f093d123-433c-4d18-9383-40ef69e48291)

## 函数计算FC功能简介
函数计算(Function Compute)

- 一个事件驱动的全托管Serverless 计算服务。
- 用户只需编写并上传代码即可运行

**FC的主要特点有：**
- 极简单开发:可在线编辑代码，随写随用。支持
NodeJS，Python，PHP，JAVA，C#，GO，Ruby等多种平台。同时可无缝接入RDS，oSS等存储产品
- 平台0配置：用户上传代码后，自动生成访问域名，自
动管理毫秒级弹性伸缩，按量计费，不执行不收费。

## 丰富的触发器功能
阿里云FC产品可以通过触发器与事件源同其他产品进行关联，设置关联后，事件源会在事件产生时以同步或异步的方式触发函数执行。为FC提供了丰富的应用空间

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/29795338-60ad-4aa7-8199-54afa45ab402)

## 函数计算FC和微服务的区别
- 相比较FC应用，通过外部访问微服务应用时，需要做较多的配置
- 而在内部访问时，微服务应用的直接访问方式效率较高

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/ef0673ca-2298-4704-be79-b26662e98f00)

## 极速上线Web应用
**利用快速几分钟搭建高可用Serverless Web应用**

- 支持常用前端开发语言。
- 可以使用在线编辑工具。
- 无需进行网络资源维护

## 事件源触机制发实现数据处理的计算存储分离
**基于事件源搭建数据处理系统**

- 调用阿里云丰富的存储服务
- 利用事件触发机制编写代码
- 无需处理流量的出入问题

## 无上限计算资源助力Al推理计算
 **让AI工程师更专注算法**
 
- 支持常用的AI推理库编写即运行
- 根据需求调用海量计算资源。
- 按量付费模式

## 毫秒级弹性扩容轻松应对突发流量
**轻松构建基于FC的高可用视频处理系统**

- 利用阿里云存储系统和触发器
- 利用高效率语言(如C)开发编解码器
- 调用海量计算资源
- 利用弹性扩容支持不可预测流量
