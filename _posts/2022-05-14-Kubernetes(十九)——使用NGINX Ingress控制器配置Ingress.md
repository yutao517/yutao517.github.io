---
layout: article
title: Kubernetes(十九)——使用NGINX Ingress控制器配置Ingress
tags: Kubernetes
category: blog
date: 2022-05-14 13:24:00 +08:00
mermaid: true
---
## 前提

部署好k8s集群，部署好ingress-nginx控制器

下面方法是使用自建的阿里云镜像部署ingress-nginx


```bash
docker pull registry.cn-hangzhou.aliyuncs.com/yutao517/ingress_nginx_controller:v1.1.0
docker tag registry.cn-hangzhou.aliyuncs.com/yutao517/ingress_nginx_controller:v1.1.0  k8s.gcr.io/ingress-nginx/controller:v1.1.1

docker pull registry.cn-hangzhou.aliyuncs.com/yutao517/kube_webhook_certgen:v1.1.1
docker tag registry.cn-hangzhou.aliyuncs.com/yutao517/kube_webhook_certgen:v1.1.1  k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
```

```bash
kubectl apply -f https://download.yutao.co/mirror/deploy.yaml
```

```bash
kubectl get svc -n ingress-nginx
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a3407f838be24eb69444bdcead2e9282.png)

```bash
kubectl get pod -n ingress-nginx
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/52b49a4a3ac34924a7f369211c3d452a.png)

部署成功ingress-nginx

## 部署一个 Hello World 应用

[官方文档](https://kubernetes.io/zh/docs/tasks/access-application-cluster/ingress-minikube/)

**创建Deployment**

```bash
vim hello-deploy.yaml
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  selector:
    matchLabels:
      run: web
  replicas: 1
  template:
    metadata:
      labels:
        run: web
    spec:
      containers:
      - name: web
        image: registry.cn-hangzhou.aliyuncs.com/yutao517/hello-app:1.0
        ports:
        - containerPort: 8080
```
```bash
kubectl apply -f hello-deploy.yaml
```
或者使用命令
```bash
kubectl create deployment web --image=registry.cn-hangzhou.aliyuncs.com/yutao517/hello-app:1.0
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f5914be79a4f460e8a9574e59abea51c.png)

**暴露服务**

```bash
vim hello-svc.yaml
```

```bash
apiVersion: v1
kind: Service
metadata:
  name: web
  labels:
    run: web
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    run: web
```

```bash
kubectl apply -f hello-svc.yaml
```

或者使用命令

```bash
kubectl expose deployment web --type=NodePort --port=8080
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/963da36dd86e4a19ad081b5f92a1c6ea.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/257cd1a57eec401db81c6cbdde52f7e5.png)

## 创建Ingress

```bash
vim hello-ingress.yaml
```

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: default
spec:
  ingressClassName: nginx
#使用nginx的IngressClass(关联的ingress-nginx控制器)
  rules:
    - host: hello-world.info
#将域名映射到web服务
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
#将所有请求发送到web服务的8080端口 
```

```bash
kubectl apply -f hello-ingress.yaml
```

```bash
kubectl get ingress
```

看到IP ADDRESS

![在这里插入图片描述](https://img-blog.csdnimg.cn/594d1da66a6b41a19b6d59a0ff2c8eb9.png)

添加域名解析规则
```bash
cat <<EOF >>/etc/hosts
10.10.121.81  hello-world.info
EOF
```

```bash
curl hello-world.info
curl 10.10.121.81
```
检验效果的不同，发现域名正常访问，但是IP无法访问，因为hello-ingress.yaml只设置了域名host的指向，没有设置IP

![在这里插入图片描述](https://img-blog.csdnimg.cn/0932b281ad21470ab1874bf1d8462a0b.png)

## 创建第二个 Deployment

```bash
kubectl create deployment web2 --image=registry.cn-hangzhou.aliyuncs.com/yutao517/hello-app:2.0
```
**暴露端口**
```bash
kubectl expose deployment web2 --port=8080 --type=NodePort
```

## 编辑现有的 Ingress

```bash
vim example-ingress.yaml 
```
追加v2路径
```bash
           - path: /v2
             pathType: Prefix
             backend:
               service:
                 name: web2
                 port:
                   number: 8080
```

```bash
kubectl apply -f example-ingress.yaml
```

```bash
curl hello-world.info
curl hello-world.info/v2
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/d2965d20c39c444b9b4fc3268ee7f3fe.png)
