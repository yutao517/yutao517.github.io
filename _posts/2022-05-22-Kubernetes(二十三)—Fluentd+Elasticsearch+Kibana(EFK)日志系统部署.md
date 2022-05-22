---
layout: article
title: Kubernetes(二十三)—Fluentd+Elasticsearch+Kibana(EFK)日志系统部署
tags: Kubernetes Project
category: blog
date: 2022-05-22 22:10:00 +08:00
mermaid: true
---

[视频详解](https://www.bilibili.com/video/BV1Qi4y1b79r?p=3)

源码地址：
[https://download.yutao.co/k8s/EFK/](https://download.yutao.co/k8s/EFK/)
## EFK

**Elasticsearch**：是一个分布式的搜索和数据分析引擎，非常适用于索引和搜索大量日志数据。
**Fluentd**：是一个开源的数据收集器，我们可以在 Kubernetes 集群节点上安装 Fluentd，通过获取容器日志文件、过滤和转换日志数据，然后将数据传递到 Elasticsearch 集群，在该集群中对其进行索引和存储。
**Kibana**：是一个免费且开放的 Elasticsearch 数据可视化Dashboard，通过web 界面进行各种操作。

## 工作原理

EFK利用部署在每个节点上的 Fluentd 采集 Kubernetes 节点服务器的 /var/log 和 /var/lib/docker/container 两个目录下的日志，然后传到 Elasticsearch 中。最后，运维人员通过访问 Kibana 来查询日志。

**生产环境**中Elasticsearch和Kibana不一定部署在K8S集群中，Elasticsearch和Fluentd之间也会部署Kafka队列做缓冲，解决Elasticsearch性能跟不上的问题，引入Kafka也使得后台升级维护变得轻松。

Fluentd使用DaemonSet控制器发布，确保全部或某些节点上运行且只运行一个Pod，所以主要用于运行存储插件、监控插件、日志插件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/487a7116cca24a2da6334623359c444a.png)
**创建命名空间**
```bash
vim logging.yml
```

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: logging
```

```bash
kubectl apply -f logging.yml
```
**部署elasticsearch**
```bash
vim elastic.yml
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elasticsearch
  namespace: logging
spec:
  selector:
    matchLabels:
      component: elasticsearch
  template:
    metadata:
      labels:
        component: elasticsearch
    spec:
      containers:
        - name: elasticsearch
          image: docker.elastic.co/elasticsearch/elasticsearch:6.5.4
          env:
            - name: discovery.type
              value: single-node
          ports:
            - containerPort: 9200
              name: http
              protocol: TCP
          resources:
            limits:
              cpu: 500m
              memory: 2Gi
            requests:
              cpu: 500m
              memory: 2Gi
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
  namespace: logging
  labels:
    service: elasticsearch
spec:
  type: NodePort
  selector:
    component: elasticsearch
  ports:
    - port: 9200
      targetPort: 9200
      nodePort: 31200
```

```bash
kubectl apply -f elastic.yml
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c82a005697c946ffa142dbb1131f82f6.png)

**部署kibana**

```bash
vim kibana.yml
```

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: logging
spec:
  selector:
    matchLabels:
      run: kibana
  template:
    metadata:
      labels:
        run: kibana
    spec:
      containers:
        - name: kibana
          image: docker.elastic.co/kibana/kibana:6.5.4
          env:
            - name: ELASTICSEARCH_URL
              value: http://elasticsearch:9200
            - name: XPACK_SECURITY_ENABLED
              value: "true"
          ports:
            - containerPort: 5601
              name: http
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: kibana
  namespace: logging
  labels:
    service: kibana
spec:
  type: NodePort
  selector:
    run: kibana
  ports:
    - port: 5601
      targetPort: 5601
      nodePort: 31601
```

```bash
kubectl apply -f kibana.yml
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/abab2c1823ea43cba31192436e729295.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/b39c2cb9b8b44650807788a978d11076.png)


**部署Fluentd**

```bash
vim fluentd-rbac.yml
```
```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: fluentd
  namespace: kube-system
rules:
  - apiGroups:
      - ""
    resources:
      - pods
      - namespaces
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: fluentd
roleRef:
  kind: ClusterRole
  name: fluentd
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: fluentd
    namespace: kube-system


```

```bash
vim fluentd-daemonset.yml
```

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
    version: v1
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
      version: v1
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
        version: v1
        kubernetes.io/cluster-service: "true"
    spec:
      serviceAccount: fluentd
      serviceAccountName: fluentd
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      containers:
        - name: fluentd
          image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
          env:
            - name: FLUENT_ELASTICSEARCH_HOST
              value: "elasticsearch.logging"
            - name: FLUENT_ELASTICSEARCH_PORT
              value: "9200"
            - name: FLUENT_ELASTICSEARCH_SCHEME
              value: "http"
            - name: FLUENT_UID
              value: "0"
            - name: FLUENTD_SYSTEMD_CONF
              value: disable
          resources:
            limits:
              memory: 200Mi
            requests:
              cpu: 100m
              memory: 200Mi
          volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers
```

```bash
kubectl apply -f fluentd-rbac.yml
kubectl apply -f fluentd-daemonset.yml
```
因为个人机器性能不佳，而	EFK对内存的消耗很高，所以无法在K8S容器中看到最终效果。
