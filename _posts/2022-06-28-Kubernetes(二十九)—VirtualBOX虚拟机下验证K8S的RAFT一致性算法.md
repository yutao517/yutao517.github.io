---
layout: article
title: Kubernetes(二十九)—VirtualBOX虚拟机下验证K8S的RAFT一致性算法
tags: Kubernetes
category: blog
date: 2022-06-28 18:34:00 +08:00
mermaid: true
---
## VirtualBOX与VMware

对比	VMware与现在使用的VirtualBOXnat模式下宿主机无法访问到虚拟机。所以必须做端口转发才能使用Moba堡垒机连接。

如下是部署一个K8s多master的高可用集群，对内部IP做了端口转发。

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/547c89b3-b8cc-483f-b231-402760e45293)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/cca8acf1-bc5f-4422-8499-8caf3ce470ce)


## 验证K8S的RAFT一致性算法


部署好3个master，3个node节点的k8s集群之后，发现当我们任意关掉一个master节点，集群还是可以正常使用。但是当只剩下一个master节点的时候确报错无法使用。


**关掉k8s-master3**

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/be359318-b17c-4b94-b986-cfd4ffb34272)


**关掉k8s-master2和k8s-master3**

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/0a80e077-c0c1-417d-997d-9585859c185d)


发现只剩下一个master节点的时候，集群出错。

---

```bash
当k8s的一致性算法RAFT，要求集群需要数量大于(n/2)的正常主节点才能提供服务（n为主节点数）
```
很显然1个节点小于3*1/2，所以不满足k8s的一致性算法RAFT，不能提供服务
.
