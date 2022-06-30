---
layout: article
title: Kubernetes(二十九)——VirtualBOX虚拟机下验证K8S的RAFT一致性算法
tags: Kubernetes
category: blog
date: 2022-06-28 18:34:00 +08:00
mermaid: true
---
## VirtualBOX与VMware

对比	VMware与现在使用的VirtualBOXnat模式下宿主机无法访问到虚拟机。所以必须做端口转发才能使用Moba堡垒机连接。

如下是部署一个K8s多master的高可用集群，对内部IP做了端口转发。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a843b9f1f9b94f5cbcb1bea3ff16e78b.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/3b4449b27ca04eed95d914126db5eee8.png)

## 验证K8S的RAFT一致性算法


部署好3个master，3个node节点的k8s集群之后，发现当我们任意关掉一个master节点，集群还是可以正常使用。但是当只剩下一个master节点的时候确报错无法使用。


**关掉k8s-master3**

![在这里插入图片描述](https://img-blog.csdnimg.cn/eb5ad48a426d40caaf0eca7c50620dcf.png)

**关掉k8s-master2和k8s-master3**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2280d1dcdcc1413cabc12af33679fe95.png)

发现只剩下一个master节点的时候，集群出错。

---

```bash
当k8s的一致性算法RAFT，要求集群需要数量大于(n/2)的正常主节点才能提供服务（n为主节点数）
```
很显然1个节点小于3*1/2，所以不满足k8s的一致性算法RAFT，不能提供服务
.
