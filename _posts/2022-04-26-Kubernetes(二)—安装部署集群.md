---
layout: article
title: Kubernetes(二)—安装部署集群
tags: Kubernetes
category: blog
date: 2022-04-26 22:14:00 +08:00
mermaid: true
---
## 安装
环境：4台服务器，1台master(192.168.2.248)，3台node(192.168.2.249~251)

**修改主机名**

每台主机都要修改
```bash
 hostnamectl set-hostname master
 hostnamectl set-hostname node1
 hostnamectl set-hostname node2
 hostnamectl set-hostname node3
```

**修改Cgroup驱动**
由于我们是在虚拟机的场景下运行，硬件资源比较紧缺，因此我们还需要修改Cgroup的驱动，将每一台主机的驱动改为systemd。
```bash
cat <<EOF> /etc/docker/daemon.json
{
"exec-opts":["native.cgroupdriver=systemd"]
}
EOF
```

```bash
systemctl restart docker
#重启
docker info | grep Cgroup
#查看驱动
```
安装完Docker执行命令docker info时如果报以下警告

> WARNING: bridge-nf-call-iptables is disabled WARNING:bridge-nf-call-ip6tables is disabled

执行命令vim /etc/sysctl.conf，在文件中追加以下内容后保存退出

```bash
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
```

**关闭swap分区**
每台主机都要关闭
```bash
swapoff -a
#临时关闭
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
#永久关闭，注释/ext/fstab/的/dev/mapper/centos-swap
cat /etc/fstab
#查看是否注释
```

```bash
su -root
```
**添加解析记录**
```bash
cat >>/etc/hosts <<EOF
192.168.2.248 master
192.168.2.249 node1
192.168.2.250 node2
192.168.2.251 node3
EOF
```
**kubelet kubeadm kubectl安装**
配置Kubernetes的yum仓库
```bash
cat >/etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

```bash
yum install kubelet kubeadm kubectl -y
```

```bash
systemctl enable kubelet
```

### 部署Kubernetes Master
准备coredns镜像
```bash
docker pull coredns/coredns
docker tag coredns/coredns registry.aliyuncs.com/google_containers/coredns
```

```bash
kubeadm init \
--apiserver-advertise-address 192.168.2.248
--image-repository registry.aliyuncs.com/google_containers \
--pod-network-cidr=10.244.0.0/16 \
--service-cidr=10.1.0.0/16 
```

> --apiserver-advertise-address 192.168.2.248 填写你自己master的IP地址

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/35d5a8337ad246bfab749d28ff672319.png)

### node节点加入K8S集群
建议不要在node节点运行kublet服务，不然会导致join失败，如果已经运行，建议执行kubeadm reset然后删除提示的文件和目录。

```bash
kubeadm join 192.168.2.248:6443 --token m5p64y.x4x1k0hthzic9ss4 \
	--discovery-token-ca-cert-hash sha256:6e43392975c729de140d8f11bb8375844461b19b3be1686f0453e64b43380fe4 
```
报错就重启docker服务，问题就会解决。

master查看node节点状态
```bash
kubectl get nodes
```
现在还没有准备好

删除节点可以使用命令
```bash
 kubectl drain node1 --delete-emptydir-data --force --ignore-daemonsets node/node1
 kubectl delete node node1
```

```bash
kubectl get pods -n kube-system -o wide
```
### master安装网络插件fanel

```bash
wget https://raw.githubusercontent.com/yutao517/code/main/kube-flannel.yml

kubectl apply -f kube-flannel.yml
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/04fbf285d5eb404eaad7ca687787ae61.png)

```bash
kubectl get nodes -n kube-system -o wide
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0a27a9ffe5914982a2da670866a4fb02.png)
