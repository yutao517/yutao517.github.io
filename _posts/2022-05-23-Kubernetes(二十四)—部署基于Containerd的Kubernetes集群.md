---
layout: article
title: Kubernetes(二十四)—部署基于Containerd的Kubernetes集群
tags: Kubernetes 
category: blog
date: 2022-05-23 20:11:00 +08:00
mermaid: true
---
## 环境

> k8s-master1：192.168.2.58
>  k8s-node1：192.168.2.158
> k8s-node2：192.168.2.159

## 部署前配置
**所有节点操作**

**关闭防火墙selinux**

```bash
 systemctl stop firewalld && systemctl disable firewalld 
```

```bash
setenforce 0 
sed -i 's/enforcing/disabled/' /etc/selinux/config 
```
**关闭swap**

```bash
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab   
```
**添加解析记录**
```bash
cat <<EOF >>/etc/hosts
192.168.2.58 k8s-master1
192.168.2.158 k8s-node1
192.168.2.159 k8s-node2
EOF
```
**部署启动时间同步服务器**

```bash
yum install chrony -y
systemctl enable chronyd
systemctl start chronyd
chronyc sources
```

## 部署Containerd
**所有节点操作**

**安装依赖常用工具**
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2 wget vim yum-utils net-tools epel-release
```
**添加加载的内核模块**

```bash
cat << EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```
**加载内核模块**

```bash
modprobe overlay
modprobe br_netfilter
```
**设置内核参数**

```bash
cat << EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```
**应用内核参数**

```bash
sysctl --system
```
**添加docker镜像源**
```bash
cat <<EOF | sudo tee /etc/yum.repos.d/docker-ce.repo
[docker]
name=docker-ce
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF
```
**安装containerd**

```bash
yum -y install containerd.io-1.4.4-3.1.el7.x86_64
```
**配置containerd**
```bash
mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml     #生成containetd的配置文件
sed -i '/runc.options/a\ SystemdCgroup = true' /etc/containerd/config.toml && \    # 修改cgroup Driver为systemd
grep 'SystemdCgroup = true' -B 7 /etc/containerd/config.toml       #查看是否修改成功
```
**镜像加速**

```bash
vim /etc/containerd/config.toml
```
修改为国内的阿里源
```bash
endpoint = ["https://registry.cn-hangzhou.aliyuncs.com" ,"https://registry-1.docker.io"]  
#修改为国内的阿里源
```
更改sandbox_image

```bash
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.2"   
#修改为国内的阿里源
```
**启动containerd服务**

```bash
systemctl enable containerd && systemctl start containerd
```
**下载镜像检测containerd是否正常**

```bash
ctr images pull docker.io/library/nginx:alpine
ctr images ls
ctr images rm docker.io/library/nginx:alpine
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/188372468dc742a9ae7e1f540c328f74.png)

## 部署Kubernetes

```bash
cat <<EOF >/etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF
```

```bash
yum install -y kubelet-1.24.0 kubeadm-1.24.0 kubectl-1.24.0
```
**设置开机自启动**
```bash
systemctl enable --now kubelet
```
**设置crictl**
```bash
cat << EOF >> /etc/crictl.yaml
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 10 
debug: false
EOF
```
**下载镜像**

```bash
crictl pull nginx:latest
```
**查看镜像**

```bash
crictl images ls
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/14425cc256384fdd90940ac5cae32ca5.png)

## 初始化集群

```bash
kubeadm config print init-defaults > kubeadm-init.yaml
```
**编辑初始化文件**
```bash
vim kubeadm-init.yaml
```

```bash
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.2.58  #自己的IP地址
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock    
  #containerd.sock文件地址
  imagePullPolicy: IfNotPresent
  name: k8s-master1   #master节点名
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
#镜像源修改为阿里
kind: ClusterConfiguration
kubernetesVersion: 1.24.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16  #添加定义pod的网段
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```
**初始化集群**
```bash
yum -y upgrade systemd  #更新systemd
```

```bash
kubeadm init --config=kubeadm-init.yaml    #初始化
```

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/6572861616ac40e3858b52341792323b.png)

在各node节点加入集群

```bash
kubeadm join 192.168.2.58:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:73406abe76199d431aa9b5aab8443be0aee7386dd3934501504fe0007a9c5e14
```
**安装网络插件**

```bash
wget https://raw.githubusercontent.com/yutao517/mirror/main/profile/kube-flannel.yml

kubectl apply -f kube-flannel.yml
```
备用flannel插件yaml文件地址
[https://download.yutao.co/mirror/kube-flannel.yml](https://download.yutao.co/mirror/kube-flannel.yml)

master查看节点状态
```bash
kubectl get nodes -o wide
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e613304e3ff743bc9fdf987b32818643.png)

可以看到CRI是containerd

**检验集群pod**

```bash
kubectl get pod -o wide -A 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/7390f25750e04dfe8f10d475f546f7bc.png)
