---
layout: article
title: Kubernetes(十三)—HPA演练
tags: Kubernetes Project
category: blog
date: 2022-05-05 14:59:09 +08:00
mermaid: true
---
## 简介
在Kubernetes 中，HorizontalPodAutoscaler 自动更新工作负载资源（例如 Deployment 或者 StatefulSet），目的是自动扩缩工作负载以满足需求。

HPA控制器通过Metrics Server的API（Heapster的API或聚合API）获取这些数据，基于用户定义的扩缩容规则进行计算，得到目标Pod副本数量。
当目标Pod副本数量与当前副本数量不同时，HPA控制器就访问Pod的副本控制器（Deployment 、RC或者ReplicaSet）发起scale操作，调整Pod的副本数量，完成扩缩容操作。

**水平扩缩**意味着对增加的负载的响应是部署更多的 Pods。 这与 “垂直（Vertical）” 扩缩不同

 **垂直扩缩**意味着将更多资源（例如：内存或 CPU）分配给已经为工作负载运行的 Pod。
[参考文档](https://kubernetes.io/zh/docs/tasks/run-application/horizontal-pod-autoscale/)



## 运行 php-apache 服务器并暴露服务

**制作镜像**
```bash
cat<<EOF >Dockerfile
FROM php:5-apache
COPY index.php /var/www/html/index.php
RUN chmod a+rx index.php
EOF
```

```bash
cat<<EOF >index.php
<?php
  $x = 0.0001;
  for ($i = 0; $i <= 1000000; $i++) {
    $x += sqrt($x);
  }
  echo "OK!";
?>
EOF
```
```bash
docker build -t hpa-example:1.0 .
```

**导出镜像**
```bash
docker save -o hpa-example hpa-example:1.0
```
SCP传到node节点，因为不知道调度到哪个node，所以传输到所有node

![在这里插入图片描述](https://img-blog.csdnimg.cn/253a603f322f46bdb23440596ee35a2f.png)
**导入镜像**
```bash
docker load -i hpa-example
#每一个节点都导入镜像
```
```bash
cat<<EOF >php-apache.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 1
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: hpa-example:1.0
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 20m
          requests:
            cpu: 10m
---
apiVersion: v1
kind: Service
metadata:
  namespace: default
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
EOF
```

```bash
kubectl apply -f php-apache.yaml
# 运行php-apache.yaml启动php-apache
```

```bash
kubectl get pods -o wide
# 查看容器启动情况
```

```bash
kubectl get svc
# 查看服务
```

```bash
kubectl delete svc php-apache
#删除ClusterIP服务
```
```bash
kubectl expose deployment php-apache  --type="NodePort" --port=8080  --target-port=80
#暴露NodePort服务
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/c9dd3b96c871489ca1b98f539cc25ca0.png)


现在集群外的局域网主机也可以通过192.168.2.249:31676访问



![在这里插入图片描述](https://img-blog.csdnimg.cn/1001a1832165488ca3c1256a30a8be59.png)

## 创建 Horizontal Pod Autoscaler
php-apache服务器已经运行，我们将通过 kubectl autoscale 命令创建 Horizontal Pod Autoscaler。
以下命令将创建一个 Horizontal Pod Autoscaler 用于控制我们上一步骤中创建的 deployment，使 Pod 的副本数量在维持在1到10之间。
大致来说，HPA 将通过增加或者减少 Pod 副本的数量（通过 Deployment ）以保持所有 Pod 的平均CPU利用率在50%以内。
```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```
查看 Autoscaler 的状态：

```bash
kubectl get hpa
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/402b8e27baa246f99ce421b987b23962.png)

**增加负载**

```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://10.244.4.43; done"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3e87cf629fb14533bceff3286d4f89aa.png)

或者局域网其他主机通过以下命令循环访问

```bash
while sleep 0.01; do wget -q -O- http://192.168.2.249:31676; done
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0bbca6ddf6bf431bb4b06b47be58d73a.png)

30s左右，通过以下命令，我们可以看到 CPU 负载升高，水平扩展了，Hpa会根据Pod的CPU使用率动态调节Pod的数量。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2b761ada6ecb4e2ebd3ff4353efbbc00.png)

ctrl +c停止负载，等待几分钟可以看到自动缩容

![在这里插入图片描述](https://img-blog.csdnimg.cn/ff66d1988fad416da809b69cc17df0db.png)
