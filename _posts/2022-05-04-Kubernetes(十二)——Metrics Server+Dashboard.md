---
layout: article
title: Kubernetes(十二)—Metrics Server+Dashboard
tags: Kubernetes
category: blog
date: 2022-05-05 13:13:09 +08:00
mermaid: true
---
## 简介

**Metrics-Server**是集群核心监控数据的聚合器。通俗地说，它存储了集群中各节点的监控数据，并且提供了API以供分析和使用。

## 部署Metrics-Serve

```bash
kubectl apply -f
https://download.yutao.co/mirror/components.yaml
```
如果部署成功下面步骤可以省略，直接跳转到部署Dashboard，这是components.yaml的内容
```bash
cat<<EOF >components.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP 
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        image: registry.aliyuncs.com/google_containers/metrics-server:v0.6.0
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 4443
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
EOF
```

```bash
kubectl apply -f metrics-server.yaml
```
检测是否安装成功
```bash
kubectl get deploy -n kube-system metrics-server
```

```bash
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
```


## 部署Dashboard
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
```
如果访问慢使用下面的个人链接

```bash
kubectl apply -f https://download.yutao.co/mirror/recommended.yaml
```
检测是否安装成功
```bash
kubectl get pod -n kubernetes-dashboard
```

```bash
kubectl patch svc kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}' -n kubernetes-dashboard
# 修改server的暴露端口方式，需要能外部访问，修改为nodeport类型
```

```bash
kubectl get svc -n kubernetes-dashboard
```
看到修改为nodeport类型，还有暴露的端口号。
![在这里插入图片描述](https://img-blog.csdnimg.cn/129efaaa2ed64fa591a06d2e81e385e5.png)

> https://192.168.2.249:31283
> 注意是https，输入自己node节点的ip和端口号

![在这里插入图片描述](https://img-blog.csdnimg.cn/10992becfda5407a8a7ac440a0cba26f.png)

**生成token**

回到master节点
```bash
cat<<EOF > admin-role.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
EOF
```

```bash
kubectl apply -f  admin-role.yaml 
```

```bash
kubectl -n kube-system get secret|grep admin-token
```

```bash
kubectl -n kube-system describe secret admin-token-txmtz
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/dae85baa5c7743abb3a9ab9ad1656040.png)

将token粘贴到web

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a5e2db56cd04ce7931db4057ef1f32f.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/619ed4811c6243bf81a6e33eada9ada0.png)

上图中的工作负载列表就是控制器

