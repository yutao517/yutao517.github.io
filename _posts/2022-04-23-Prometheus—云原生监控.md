---
layout: article
title: Prometheus—云原生监控
tags: Prometheus
category: blog
date: 2022-04-24 22:17:00 +08:00
mermaid: true
---
## 简介
**监控与运维的作用**
- 能够对系统进行7*24小时的实时监控 
- 能够及时反馈系统状态 
- 保证平台的稳定运行 
- 保证服务的安全可靠
- 保证业务的持续运行

**概况**

Prometheus是用于事件监控和警报的免费软件应用程序。该项目是用Go编写的，并在 Apache 2 许可下获得许可，源代码可在GitHub 上获得，用于云原生计算的监控，2016 加入云原生计算基金会，成为继Kubernetes（K8S）之后的第二个托管项目。

**架构图**

![在这里插入图片描述](https://img-blog.csdnimg.cn/10c6510aa2d146c2b4de15ee62f32ae8.png)

- Prometheus Server：服务端用于抓取指标、存储时间序列数据
- exporter：采集数据，并通过HTTP 服务的形式暴露给Prometheus Server，Prometheus Server通过访问该Exporter 提供的接口，即可获取到需要采集的监控数据
- pushgateway：push 的方式将指标数据推送到该网关
- alertmanager：告警管理器，处理报警的报警组件
- adhoc：用于数据查询
- grafana：图形展示工具

**pull和push两种方式**
- pull具有可控性，可控服务器的负载，网络的负载
- push保证数据的实时性，得到最flush的数据

## 安装
**Prometheus Server**

**容器方式启动**

```bash
docker run --name prometheus -dp 9090:9090 prom/prometheus
```
安装成功后访问9090端口显示获取服务器时间时出错：检测到浏览器和服务器之间有时间差Prometheus 依赖于准确的时间，时间漂移可能会导致意外的查询结果，所以我们调整服务器为网络时间。

![在这里插入图片描述](https://img-blog.csdnimg.cn/73e9a1d599d0433ab37a4793feb91f3e.png)

```bash
yum install ntp ntpdate -y
#安装ntpdate工具
ntpdate cn.pool.ntp.org
#设置系统时间与网络时间同步
hwclock --systohc
#将系统时间写入硬件时间
clock -w
#强制写入CMOS防止重启失效
```

```bash
http://192.168.2.249:9090
#输入自己服务器IP地址
```
**主机方式启动**

换一台服务器
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.35.0/prometheus-2.35.0.linux-amd64.tar.gz
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/5e2f8cd87b9540728099cf67fafdab7f.png)

```bash
tar xvfz prometheus-*.tar.gz -C /usr/local/
mv /usr/local/prometheus-* /usr/local/prometheus
echo  "PATH=$PATH:/usr/local/prometheus/" >>/root/.bashrc
source /root/.bashrc
cd /usr/local/prometheus/
nohup prometheus & 
#回车后可在后台运行，不占用终端，输出默认在在当前目录nohup.out
```

```bash
http://192.168.2.248:9090
```

 **exporter**
换一台被监控机安装node_exporter，我们这里通过node-exporter来获取node节点信息，node_exporter 就是用于采集服务器节点的各种运行指标的，
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar xvfz  node_exporter-1.3.1.linux-amd64.tar.gz -C /usr/local
mv /usr/local/node_exporter-1.3.1.linux-amd64 /usr/local/node_exporter
echo  "PATH=$PATH:/usr/local/node_exporter/" >>/root/.bashrc
nohup node_exporter &
```

```bash
http://192.168.2.250:9100/
#访问自己服务器地址的9100端口
```
server通过webhook技术去访问这个网址获取node上的metrics

## 配置
**浏览器访问服务端IP地址**

```bash
http://192.168.2.248:9090
```
Status--->Targets

可以看到只监控服务器本机

**更改server配置文件**

```bash
vim /usr/local/prometheus/prometheus.yml 
```
末尾追加
```bash
 - job_name: "node"
    static_configs:                  
      - targets: ["192.168.2.250:9100"]
```
因为Prometheus没有服务重启，所以只能
查看进程杀死再启动

![在这里插入图片描述](https://img-blog.csdnimg.cn/a6c93a5d496e48d7a7269537dd3678fc.png)

刷新server web看到已经启动监控

![在这里插入图片描述](https://img-blog.csdnimg.cn/7358c9ffe07949b3be3752c1b8b14e9a.png)

Graph查找CPU可以看到相关数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/583a03659e364c4ea7b9a6ad5bb54f74.png)
