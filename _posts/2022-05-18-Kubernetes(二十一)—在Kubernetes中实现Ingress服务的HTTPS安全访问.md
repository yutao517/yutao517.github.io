---
layout: article
title: Kubernetes(二十一)—在Kubernetes中实现Ingress服务的HTTPS安全访问
tags: Kubernetes Project
category: blog
date: 2022-05-18 14:22:00 +08:00
mermaid: true
---
## 开启HTTPS服务

```bash
vim nginx-service.yaml
```

暴露443端口
```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx2-pv
  labels:
    run: nginx2-pv
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    name: http
  - port: 443
    targetPort: 443
    nodePort: 30443
    name: https
  selector:
    run: nginx2-pv
```

```bash
kubectl apply -f nginx-service.yaml
```

## 创建集群的服务器证书

```bash
mkdir /root/cert &&cd /root/cert
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/25db62b09d4747abb08d252aafc2f8c4.png)

## Ingress上配置证书

```bash
kubectl create secret tls secret-https --key tls.key --cert tls.crt     
```

```bash
kubectl describe secret secret-https
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0082844cb45f4541abd98ed8a875606e.png)

## 开启TLS
增加tls规则
```bash
  tls:
  - hosts: 
    - hello-world.info
    secretName: secret-https 
```

```bash
cat nginx-ingress.yaml 
```

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pv-nginx
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts: 
    - hello-world.info
    secretName: secret-https 
  rules:
  - host: hello-world.info
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx2-pv
            port:
              number: 80

```

```bash
kubectl apply -f nginx-pv-ingress.yaml
```

## 检验HTTPS

```bash
kubectl get svc -n ingress-nginx
kubectl get ingress
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/4a729598990c404fb0cd04aac5f0dc02.png)

因为使用的自签名，所以会提示不被信任，所以在实际生产环境，需要去阿里云注册签名。

> https://hello-world.info
> 
>![在这里插入图片描述](https://img-blog.csdnimg.cn/78209e8e25624e9887f608fb02e8a6a5.png)
