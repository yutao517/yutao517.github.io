---
layout: article
title: Kubernetes(二十五)—Containerd容器镜像管理
tags: Kubernetes 
category: blog
date: 2022-05-26 11:55:00 +08:00
mermaid: true
---
## Containerd容器镜像管理命令
- docker使用docker images命令管理镜像
- 单机containerd使用ctr images命令管理镜像
- k8s集群中containerd使用crictl images命令管理镜像

**拉取镜像**
```bash
ctr images pull --all-platforms docker.io/library/nginx:alpine
```
下载所有平台架构的nginx:alpine镜像

```bash
ctr images pull --platform linux/amd64 docker.io/library/nginx:latest
```
下载linux/amd64架构的nginx:latest镜像

![在这里插入图片描述](https://img-blog.csdnimg.cn/8b7c975eca9343c89c5ebd79cb5c4f1c.png)

```bash
ctr images ls   #列出镜像
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/bb8f99e9974041f287d496bd4be76e95.png)

**删除镜像**

```bash
ctr images rm docker.io/library/nginx:alpine    #删除镜像nginx:alpine
ctr images rm $(ctr images ls |awk 'NR>1{print $1}')
#删除所有镜像
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/7ce6a1f05900486b8f39836c71b9de1b.png)
**挂载镜像**

```bash
ctr images mount docker.io/library/nginx:latest /mnt
#将nginx镜像挂载到/mnt目录
```

```bash
ls /mnt
```

```bash
ls /mnt/usr/share/nginx/html
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b90391936e05440fa7bae9b4e15e8679.png)

```bash
umount /mnt   #解挂镜像
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f4cb381b18c049628da98801b380e02a.png)

**镜像导出导入**

```bash
ctr images export nginx.img  docker.io/library/nginx:latest 
#镜像导出nginx:latest名为nginx.img
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/416b27abe69e46f98052e5afce17cce1.png)

```bash
ctr images import nginx.img 
#镜像导入nginx
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a48220322f764fd681717d20485b6e2e.png)

**修改镜像tag**

```bash
ctr images tag docker.io/library/nginx:latest yutao.co/library/nginx:latest
```

```bash
ctr images check
#镜像检查
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/27d1b090cfed45ebbb326e67f297e34d.png)

**查看容器**

```bash
ctr containers ls
ctr c ls
```
**查看运行的容器(任务)**

```bash
 ctr task ls
 ctr t ls
```
**创建容器**

```bash
ctr images pull docker.io/library/nginx:latest
ctr containers create docker.io/library/nginx:latest my-nginx
ctr containers ls
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/9dc4b8507a924465bd87bbceff756c91.png)

**查看容器详细信息**
类似docker中的inspect
```bash
ctr containers info my-nginx
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/96b434215c0f4a95917cd39dabed3c67.png)

**启动容器**

```bash
ctr task start -d my-nginx
# -d在后台运行
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/3556eb524839467d89cff5cd23c9eb7d.png)

**进入容器**
```bash
ctr tasks exec --exec-id 0 my-nginx sh
```
**运行动态容器**

```bash
ctr run -d --net-host docker.io/library/nginx:alpine nginx1
ctr t exec --exec-id $RANDOM -t  nginx1 sh
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c15cec094aad4082abdbc5d88f76ccf3.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/eaa5d5ef19f044fc9a3b5e80ea622356.png)

**暂停容器**

```bash
ctr task pause my-nginx3
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/eedc953550334da395991e77ff7c51df.png)

**恢复容器**

```bash
ctr task resume my-nginx3
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/96735059c91c4cf28be8d5c248363304.png)

**停止容器**

```bash
ctr task kill my-nginx3
#停止容器
ctr t rm my-nginx3
#删除容器进程
ctr containers delete my-nginx3
ctr c rm my-nginx3
#删除容器
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/3b4f2a201e2245bc85aebfefcccccc14.png)
