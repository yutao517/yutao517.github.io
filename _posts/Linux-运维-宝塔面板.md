

## 一.安装要求：

内存：512M以上，推荐768M以上（纯面板约占系统60M内存）
硬盘：300M以上可用硬盘空间（纯面板约占20M磁盘空间）
系统：CentOS 7.1+ (Ubuntu16.04+.、Debian9.0+)，确保是干净的操作系统，没有安装过其它环境带的Apache/Nginx/php/MySQL/pgsql/gitlab/java（已有环境不可安装）
架构：x86_64（主流服务器均是此架构），ARM不完整兼容（面板环境安装慢，部分软件可能安装不上）

## 二.安装命令：
**Centos安装命令**
```bash
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```
**Ubuntu/Deepin安装命令**

```bash
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh
```
**Debian安装命令：**

```bash
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && bash install.sh
```
**Fedora安装命令:**

```bash
wget -O install.sh http://download.bt.cn/install/install_6.0.sh && bash install.sh
```
**Linux面板7.8.0升级命令：**

```bash
curl http://download.bt.cn/install/update6.sh|bash
```

## 三.登录宝塔面板
![在这里插入图片描述](https://img-blog.csdnimg.cn/2a7fe8db91e449809bbc542d0f09b054.png)
网页登录外网地址即可打开，如果存在打不开的情况，进入所属服务器控制台打开安全组配置开放8888端口。
以华为云服务为例，进入华为云官网云服务器控制台，
点击更多，更改安全组。

![在这里插入图片描述](https://img-blog.csdnimg.cn/0a1fb0bda1da41d399cc20cc1917eab1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_14,color_FFFFFF,t_70,g_se,x_16)
新建安全组
![](https://img-blog.csdnimg.cn/a830a59fd8674908af6382c513a5bec9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
sys-webserver配置规则
![在这里插入图片描述](https://img-blog.csdnimg.cn/d0d546685c39424888094911da75a91b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
左上角添加规则
![在这里插入图片描述](https://img-blog.csdnimg.cn/2d1d8a9f520143bcac29173c6f2682e5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
优先级填1，TCP端口选择开放的端口8888，点击确定，刷新完成。
![在这里插入图片描述](https://img-blog.csdnimg.cn/13e0803dd7a54ddd9dd27a7ed03437eb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
输入登录用户密码，就是安装好显示的username和password，如果忘记，进入ssh终端输入bt命令查看14
![在这里插入图片描述](https://img-blog.csdnimg.cn/a4154aa1d9b840bb8f13c3536ad1a728.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9837a61d00074b07b68a00df26aa48f3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)


## 四.面板特色功能：

一键配置服务器环境（LAMP/LNMP）
一键安全重启
一键创建管理网站、ftp、数据库
一键部署SSL证书
一键部署源码（discuz、wordpress、dedecms、z-blog、微擎等等）
一键配置（定期备份、数据导入、伪静态、301、SSL、子目录、反向代理、切换PHP版本）
一键安装常用PHP扩展(fileinfo、intl、opcache、imap、memcache、apc、redis、ioncube、imagick)
数据库一键导入导出
系统监控（CPU、内存、磁盘IO、网络IO）
防火墙端口放行
SSH开启与关闭及SSH端口更改
禁PING开启或关闭
方便高效的文件管理器（上传、下载、压缩、解压、查看、编辑等等）
计划任务（定期备份、日志切割、shell脚本）
软件管理（一键安装、卸载、版本切换）
