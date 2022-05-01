---
layout: article
title: Kubernetes(九)—NodePort类型Service暴露应用
tags: Kubernetes
category: blog
date: 2022-05-01 15:52:00 +08:00
mermaid: true
---
## Service资源
尽管每个 Pod 都有一个唯一的 IP 地址，但是如果没有 Service ，这些 IP 不会暴露在集群外部。

- ClusterIP：在集群的内部 IP 上公开 Service 。这种类型使得 Service 只能从集群内访问。
- NodePort：使用 NAT 在集群中每个选定 Node 的相同端口上公开 Service
- LoadBalancer： 在当前云中创建一个外部负载均衡器(如果支持的话)，并为 Service 分配一个固定的外部IP
- ExternalName： 通过返回带有该名称的 CNAME 记录，使用任意名称公开 Service。


[参考文档](https://kubernetes.io/zh/docs/concepts/services-networking/connect-applications-service/)
## 在集群中暴露 Pod 
创建pod
```bash
vim my-nginx.yaml
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 6
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f my-nginx.yaml
```

```bash
kubectl get pods -o wide
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/8d4f3742fbbf4a11988d00f32bf11b23.png)


**NodePort类型暴露Service**

```bash
vim service.yaml
```

```bash
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    run: my-nginx
```

```bash
kubectl apply -f servcie.yaml
```

```bash
kubectl get service my-nginx
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6c1e2bd579494b7a8e20518c617afdaf.png)

> 可以看到8080:31944 
> 8080其实是k8s集群内部的一个VIP，做负载均衡
> 31944其实是宿主机master的端口

 我们查看一下服务的详细信息

```bash
 kubectl describe service my-nginx
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7d07120255944b57b6f61190e4b2f064.png)
所以说8080端口是VIP10.1.132.180的端口

我们进入容器内部把显示的页面更改，查看负载均衡的效果。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d10ec14a4eaa4fc5a51485cd57a66f39.png)

我们多次访问VIP的8080端口可以看到负载均衡效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/cf4a2c0545624a8eaadd8577b51b7087.png)

使用集群外的同网段机器访问宿主机的31944端口，可以看到负载均衡效果。

![在这里插入图片描述](https://img-blog.csdnimg.cn/4a35d78d712b404d8f3803bde0bf4208.png)

当然我们也可以使用**命令暴露服务**

```bash
kubectl delete service my-nginx
#先删除服务
```

```bash
kubectl expose deployment/my-nginx --type="NodePort" --port=8080  --target-port=80 --name=my-nginxservice/my-nginx exposed
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/bedadab72e5f4c3f98c4554bd95ab74c.png)

可以看到又随机生成一个VIP，宿主机的代理端口为32747
所以NodePort类型在集群中的主机节点上为Service随机提供一个代理端口，以允许从主机网络上对Service进行访问。
