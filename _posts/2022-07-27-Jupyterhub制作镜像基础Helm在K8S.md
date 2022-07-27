---
layout: article
title: Jupyterhub制作镜像基础Helm在K8S
tags: Jupyterhub
category: blog
date: 2022-07-27 17:02:00 +08:00
mermaid: true
---
# 无服务平台

**微软AZURE的Azure Functions**

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb5db8d1d86744bfb60e583ddbabf92c.png)

阿里云函数计算 FC

![在这里插入图片描述](https://img-blog.csdnimg.cn/3e9f36bc5f334ed3a5d227f0ac376a0f.png)

腾讯云Serverless

![在这里插入图片描述](https://img-blog.csdnimg.cn/b6089080b01b4aaf912fc91ae39bc5dd.png)
有终端

华为云函数工作流

![在这里插入图片描述](https://img-blog.csdnimg.cn/0b0a16ae9dd34314bfd896ec5c384c92.png)

无终端

# 主机安装配置

```bash
mkdir jupyterhub && cd jupyterhub
yum install python3 -y
python3 -m pip install --upgrade pip
yum update
yum install npm nodejs-legacy
(apt-get install libnode64)
npm install n -g
n lts
pip install notebook -i https://pypi.douban.com/simple
pip install jupyterlab -i https://pypi.douban.com/simple
npm install -g configurable-http-proxy
python3 -m pip install jupyterhub
jupyterhub --generate-config 
```

```bash
vim jupyterhub_config.py
```

```bash
#c.Spawner.default_url = '/lab'
# /lab对应jupyterlab 默认为notebook
c.JupyterHub.port = 80
# 指定暴露端口
c.PAMAuthenticator.encoding = 'utf8'
c.Authenticator.whitelist = {'root','admin', 'jupyter', 'aiker'}
# 指定可使用用户
c.LocalAuthenticator.create_system_users = True
c.Authenticator.admin_users = {'root', 'admin'}
# 指定admin用户
c.JupyterHub.statsd_prefix = 'jupyterhub'
c.Spawner.notebook_dir = '/volume1/study/' 
#jupyterhub自定义目录
c.Spawner.cmd=['jupyterhub-singleuser']
c.JupyterHub.port = 443
# 更换端口为443
# 证书从阿里云安装到本地/root/jupyterhub
c.JupyterHub.ssl_cert = 'jupyterhub.yutao.pem'
c.JupyterHub.ssl_key = 'jupyterhub.yutao.key'
```

```bash
useradd admin
passwd admin
mkdir -p /volume1/study/
chown admin /volume1/study/ -R
chown admin /home/admin -R
```
**启动**
```bash
jupyterhub
```

将域名解析到本机服务器

```bash
https://jupyterhub.yutao.co/
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/3ec349cdfdc746199ddb264bfb3525e3.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/dca796cfd11d47c78e1ef70c0e2f8002.png)


```bash
pip install jupyterlab-language-pack-zh-CN
#中文汉化
```

## 编写启动服务

```bash
vim /usr/local/bin/jupyterhub.sh
```

```bash
#!/bin/bash
cd /root/jupyterhub
jupyterhub
```

```bash
chmod 777 jupyterhub.sh
```

```bash
vim /etc/systemd/system/jupyterhub.service
```

```bash
[Unit]
Description=jupyterhub
After=network.target

[Service]
User=root
ExecStart=/usr/local/bin/jupyterhub.sh

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl start jupyterhub
systemctl enable jupyterhub
systemctl status jupyterhub
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/7de1a9c5aff342a8a7a96865ebf40315.png)

```bash
ps aux|grep jupyterhub
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cad16076d83c49049b504893dc693b5b.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c8f16fab3476435ea1bf25f51a97119e.png)

# K8S

首先保证有一个K8S集群的环境
![在这里插入图片描述](https://img-blog.csdnimg.cn/49ba2d84290543fbbc2ba1c2ea03d2ae.png)


## 创建一个Deployment

**jupyterhub.yaml**
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jupyterhub
spec:
  selector:
    matchLabels:
      run: jupyterhub
  replicas: 1
  template:
    metadata:
      labels:
        run: jupyterhub
    spec:
      containers:
      - name: jupyterhub
        image: registry.cn-hangzhou.aliyuncs.com/yutao517/jupyterhub
        ports:
        - containerPort: 8000
```

## 创建一个NodePort类型的service暴露端口

**jupyterhub-svc.yaml**

```bash
apiVersion: v1
kind: Service
metadata:
  name: jupyterhub
  labels:
    run: jupyterhub
spec:
  type: NodePort
  ports:
  - port: 8000
    targetPort: 8000
    nodePort: 31080
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    nodePort: 30443
    name: https
  selector:
    run: jupyterhub
```

## 创建一个Ingress服务，并开启HTTPS功能

**创建集群的服务器证书**

```bash
mkdir /root/cert &&cd /root/cert
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a6dbd28b995244468a31b262fb2abf0a.png)


**Ingress上配置证书**

```bash
kubectl create secret tls secret-https --key tls.key --cert tls.crt     
kubectl describe secret secret-https
```

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: jupyterhub
  namespace: default
spec:
  tls:
  - hosts:
    - hub.jupyter.com
    secretName: secret-https
  ingressClassName: nginx
  rules:
    - host: hub.jupyter.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: jupyterhub
                port:
                  number: 8000
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/35b57c7b6b5140f2a1e2fa1f03ba2520.png)
**配置DNS解析**

![在这里插入图片描述](https://img-blog.csdnimg.cn/bec3f8d580f3443992d97d4b06ac984d.png)

## 通过域名进行访问
因为使用的是K8S的自签名，所以会提示不安全。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac1bb138ce38433a9b22cf14917a46ad.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/af0b6d2c7dff4c57983ee64b9c81ffb4.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/1572963beff44b84b59f49842f071582.png)

因为是自签名的HTTPS所以看到不安全，但是已经配置好HTTPS。没有配置HTTPS如下图所示：


![在这里插入图片描述](https://img-blog.csdnimg.cn/030693c2b6434087aa5f3e1c3aecf8f6.png)

登录，去容器新建用户，因为默认不允许root用户登录。

```bash
kubectl exec -it jupyterhub-6768ddf8cd-pjsm8 -- /bin/bash
useradd yutao_wang
passwd user1
mkdir /home/yutao_wang
cd /home
chown yutao_wang  yutao_wang -R
su yutao_wang
pip install jupyterlab -i https://pypi.douban.com/simple
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/39e348335564464f8afee8c2bf09bd82.png)



![在这里插入图片描述](https://img-blog.csdnimg.cn/37fc23f9e9084138a11e3e243ead3c99.png)


正在改进。做数据固化，不然只能启一个副本。

## HELM 使用
```bash
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update
```

```bash
openssl rand -hex 32
```

```bash
vim config.yaml
```

```bash
proxy:
  secretToken: "33ad66059c5ae9dbf63576c82489b4b5cfaed27141d1be515c7f7b2f72756a92" #自己的密钥
hub:
  db:
    type: sqlite-memory
singleuser:
   storage:
      type: none
```

```bash
vim helm-jypyterhub.sh
```
```bash
RELEASE=jhub
NAMESPACE=jhub
helm upgrade --cleanup-on-fail \
  --install $RELEASE jupyterhub/jupyterhub \
  --namespace $NAMESPACE  \
  --create-namespace \
  --version=1.1.3 \
  --values config.yaml \
  --timeout 600s
```
```bash
./helm-jypyterhub.sh
```
出现镜像拉取错误，阿里云重新拉取并打标签
```bash
docker pull  registry.cn-hangzhou.aliyuncs.com/yutao517/pause:3.5
docker tag registry.cn-hangzhou.aliyuncs.com/yutao517/pause:3.5  k8s.gcr.io/pause:3.5
docker pull registry.cn-hangzhou.aliyuncs.com/yutao517/kube-scheduler:v1.19.13
docker tag registry.cn-hangzhou.aliyuncs.com/yutao517/kube-scheduler:v1.19.13 k8s.gcr.io/kube-scheduler:v1.19.13
```
修改proxy-public svc 将 type: LoadBalancer 改成 type: NodePort
```bash
kubectl edit service proxy-public -n jhub
```
调度到master出现容忍错误
```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

jupyterhub的默认用户：bi:bi

![在这里插入图片描述](https://img-blog.csdnimg.cn/6da06b1625714d499e213b8e4368eae2.png)


## 制作镜像
```bash
FROM registry.cn-hangzhou.aliyuncs.com/yutao517/ubuntu:focal
ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai
RUN mkdir -p /jupyterhub
WORKDIR /jupyterhub
RUN set -ex; \
    cd /jupyterhub &&\
    apt-get update &&\
    apt-get install wget -y &&\
    apt-get install curl -y &&\
    apt-get install python3-pip -y &&\
    python3 -m pip install --upgrade pip &&\
    apt-get install npm -y &&\
    apt-get install nodejs -y &&\
    apt-get install libnode64 &&\
    npm install n -g &&\
    pip install notebook &&\
    pip install jupyterlab &&\
    pip install jupyterlab-language-pack-zh-CN &&\
    npm install -g configurable-http-proxy &&\
    python3 -m pip install jupyterhub &&\
    useradd admin &&\
    echo admin:admin | chpasswd &&\
    mkdir /home/admin  &&\
    chown admin /home/admin -R 
COPY jupyterhub_config.py /jupyterhub
COPY jupyterhub.yutao.co.key /jupyterhub
COPY jupyterhub.yutao.co.pem /jupyterhub
EXPOSE 443
CMD ["jupyterhub"]
```
