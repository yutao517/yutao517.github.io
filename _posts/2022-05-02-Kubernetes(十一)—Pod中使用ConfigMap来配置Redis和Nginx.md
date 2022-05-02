---
layout: article
title: Kubernetes(十一)—Pod中使用ConfigMap来配置Redis和Nginx
tags: Kubernetes
category: blog
date: 2022-05-02 18:23:09 +08:00
mermaid: true
---
## 简介
很多应用在其初始化或运行期间要依赖一些配置信息。大多数时候， 存在要调整配置参数所设置的数值的需求。 ConfigMap 是 Kubernetes 用来向应用 Pod 中注入配置数据的方法。
ConfigMap与 Secret 类似，用来存储配置文件的kubernetes资源对象，所有的配置内容都存储在etcd中。

[参考文档](https://kubernetes.io/zh/docs/tutorials/configuration/configure-redis-using-configmap/)
## 配置 Redis

```bash
cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
EOF
```

```bash
kubectl apply -f example-redis-config.yaml 
```

```bash
cat <<EOF >redis.yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    command:
      - redis-server
      - "/redis-master/redis.conf"
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: example-redis-config
        items:
        - key: redis-config
          path: redis.conf
EOF
```

```bash
kubectl apply -f redis.yaml 
```

```bash
kubectl exec -it redis -- redis-cli
进入容器
```


```bash
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "0"
127.0.0.1:6379> config get maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
```
可以看到当前配置
退出终端追加配置

```bash
cat<<EOF >>example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru    
EOF
```

重新启动pod
```bash
kubectl delete pod redis
```
```bash
kubectl apply -f redis.yaml 
```

```bash
kubectl exec -it redis -- redis-cli
```

```bash
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "2097152"
127.0.0.1:6379> config get maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"
```
可以看到配置生效

## 配置Nginx
准备一个nginx.conf文件

```bash
cat<<EOF >nginx.conf
worker_processes  4;
events {
    worker_connections  2048;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
	listen  80;
	server_name www.yutao.co;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
EOF
```

```bash
kubectl create configmap wyt-nginx-1 --from-file=nginx.conf
#创建configmap
```

```bash
kubectl get configmap wyt-nginx-1 -o yaml
#查看configmap的内容
```
```bash
cat<<EOF >wyt-nginx.yaml
kind: Deployment
metadata:
  name: wyt-nginx
spec:
  replicas: 2
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wyt-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wyt-nginx
  template:
    metadata:
      labels:
        app: wyt-nginx
    spec:
      containers:
        - name: nginx
          image: "nginx:latest"
          imagePullPolicy: IfNotPresent
          ports:
          - containerPort: 80
          volumeMounts:
          - name: wyt-nginx-config
            mountPath: /etc/nginx/nginx.conf
            subPath: nginx.conf
      volumes:
        - name: wyt-nginx-config
          configMap:
            name: wyt-nginx-1
            items:
            - key: nginx.conf
              path: nginx.conf              
EOF
```

```bash
kubectl apply -f wyt-nginx.yaml 
```
```bash
kubectl exec -it wyt-nginx-f5f4b9b55-4h9wx  -- bash
```

```bash
cat /etc/nginx/nginx.conf |grep worker_processes
```
可以看到使用的是我们创建的配置worker_processes  4

![在这里插入图片描述](https://img-blog.csdnimg.cn/96017b433918483a883473d4e6440c90.png)
