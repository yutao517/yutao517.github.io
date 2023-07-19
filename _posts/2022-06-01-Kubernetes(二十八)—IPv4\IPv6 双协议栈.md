---
layout: article
title: Kubernetes(二十八)—IPv4\IPv6 双协议栈
tags: Kubernetes
category: blog
date: 2022-06-01 18:34:00 +08:00
mermaid: true
---
## 虚拟机配置IPV6
环境：虚拟机使用NAT模式并启用IPV6

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/6090aeff-3a4a-407b-b4e8-5d888d015797)

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33 
```

```bash
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=no
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6ADDR=fd15:4ba5:5a2b:1008::10/64
IPV6_DEFAULTGW=fd15:4ba5:5a2b:1008::1
NAME=ens33
DEVICE=ens33
IPADDR="192.168.136.58"
PREFIX=24
GATEWAY=192.168.136.2
DNS1=114.114.114.114
ONBOOT=yes
```

```bash
 vim /etc/sysctl.conf 
```

```bash
net.ipv6.conf.all.accept_dad = 0
net.ipv6.conf.default.accept_dad = 0
net.ipv6.conf.ens33.accept_dad = 0
```

```bash
sysctl -p
serice network restart
```

因为我的外网并没有IPV6功能，所以仅能在NAT局域内部使用
使用主机ping -6

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/f302b509-c97b-4c09-a3eb-3e6c4aeaf2ad)

配置成功
## sysctl参数启用ipv6
各节点都开启
```bash
vim /etc/sysctl.d/k8s.conf
```

```bash
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
fs.may_detach_mounts = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720
 
 
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
 
 
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
net.ipv6.conf.all.forwarding = 1
```

```bash
reboot
```
## 初始化集群
```bash
docker pull coredns/coredns
docker tag coredns/coredns registry.aliyuncs.com/google_containers/coredns
```

```bash
vim kubeadm-init.yaml 
```

```bash
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: fd15:4ba5:5a2b:1008::10
  bindPort: 6443
nodeRegistration:
  taints:
  - effect: PreferNoSchedule
    key: node-role.kubernetes.io/master
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.23.0
imageRepository: registry.aliyuncs.com/google_containers
networking:
  podSubnet: 10.244.0.0/16,2001:db8:42:0::/56
  serviceSubnet: 10.96.0.0/16,2001:db8:42:1::/112
```

```bash
kubeadm init --config=kubeadm-init.yaml
```

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/7940762f-d4c1-400b-8c9b-7c1abd29f094)

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
**node节点加入master**

```bash
kubeadm join [fd15:4ba5:5a2b:1008::10]:6443 --token a7nn29.3s6b1q0pjrxxuwhi \
        --discovery-token-ca-cert-hash sha256:5648eeecff97b13cb22e8d1f0ed8bbec06425424934e7afe8b90e50871e5dc7b 
```
![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/9f2dcf89-4e23-48c3-bb80-f79650cc90e3)

## 安装Calico
**卸载flannel**

```bash
#每个节点都要执行
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
rm -f /etc/cni/net.d/*
systemctl restart kubelet
```

**安装Calico**

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/15b66cd5-e981-455c-9a10-e4fc5870a535)

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/16eab2f8-a10d-4052-9e72-81cf08ad6fbd)

安装Calio-IPV6

```bash
wget https://raw.githubusercontent.com/cby-chen/Kubernetes/main/yaml/calico-ipv6.yaml
```

[备用地址：https://download.yutao.co/mirror/calico-ipv6.yaml](https://download.yutao.co/mirror/calico-ipv6.yaml)

```bash
vim calico-ipv6.yaml
```
在38行左右，改为自己的初始化podIP地址

```bash
          "ipam": {
              "type": "calico-ipam",
              "assign_ipv4": "true",
              "assign_ipv6": "true"
          },
          - name: IP
            value: "autodetect"

          - name: IP6
            value: "autodetect"

          - name: CALICO_IPV4POOL_CIDR
            value: "10.244.0.0/16"

          - name: CALICO_IPV6POOL_CIDR
            value: "2001:db8:42:0::/56"

          - name: FELIX_IPV6SUPPORT
            value: "true"
```

```bash
kubectl  apply -f calico-ipv6.yaml 
```

## 验证 IPv4/IPv6 双协议栈

```bash
kubectl get nodes k8s-node1 -o go-template --template='{{range .spec.podCIDRs}}{{printf "%s\n" .}}{{end}}'
```

![image](https://github.com/yutao517/yutao517.github.io/assets/62100249/5be69ae0-c87b-41b4-9736-2db7fa863bf6)

