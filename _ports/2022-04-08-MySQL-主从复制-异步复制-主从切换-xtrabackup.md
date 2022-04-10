---
layout article
title MySQL-主从复制-异步复制-主从切换-xtrabackup
tags MySQL数据库
category blog
date 2022-04-08 000000 +0800
mermaid true
---
## 主从复制的原理图

![在这里插入图片描述](https://img-blog.csdnimg.cn/e6d7ffd19e6442c2967ed415056f678c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

**主从复制**：其实就是使主库和从库数据一致，主库的数据变更，需要同步到从库中。

**读写分离**：如果是写操作(insert、update、delete)，则直接操作主库；如果是读(select)操作，则直接操作从库(从库是可以有多个的)
## 使用主从复制原因(目的)

- 为数据安全，做备份恢复。
- 增加服务器的数量，读写分离，减轻主库负载。
- 当主服务器出现问题时，可以主从切换，切换到从服务器，提高并发性能，实现高可用。
## 主从复制模式

- **异步复制**：下面常见的主从复制是异步模式，MySQL的复制默认是异步的，数据有延迟，导致数据丢失不一致。
- **半同步复制**
- **全同步复制（组复制）**

## 主从复制的基本原理

如图所示，有两台服务器，一台master，一台slave初始数据一样：
首先master开启二进制日志(binary log)，只要master数据变化，就将改变记录到二进制日志，master上面有一个dump线程，当二进制日志发生变化，通知slave服务器上的I/O线程来读master上的二进制日志，slave 将 master 的 二进制日志(binary log ) 拷贝到它的中继日志 (relay log) ，当中slave上还有一个sql线程，读取中继日志，并执行中继日志中的事件，使得slave上的数据和master数据一样，从而达到主从复制。

## 问题

主服务器如何知道从服务器上有哪些数据，需要从哪里开始给从服务器数据（从服务器如何知道从哪里开始获取服务器数据）

**master-info文件的作用**

- I/O线程更新master.info文件，记录主从复制的用户名密码，二进制日志文件的名字，二进制日志的pos位置号

**relay-log.info文件的作用**

- SQL线程更新relay-log.info文件，记录了 relay log 文件的进度情况

## 实验步骤
环境：有两台服务器，一台master（192.168.119.191），一台slave（192.168.119.225），保证主从服务器MySQL版本一致数据一致，在从服务器脚本直接安装MySQL。

1.主服务器master开启二进制日志,从服务器slave区分server-id =2
```bash
#配置文件[mysqld]新增两行打开
log_bin
server_id = 1 #区别服务器ID，从服务器也要写server-id =2，但是可以不加log_bin
service mysqld restart #重启mysql服务
```

2.保证主从服务器数据一致

```bash
mysqldump --all-databases >/backup/all_db.sql #备份主服务器所有数据
scp all_db.sql root@192.168.119.225 :/root #scp传输主服务器的备份到从服务器root目录下
```
3.master创建一个可以有复制权限的授权用户，这样slave可以来master复制二进制文件
```bash
mysql>grant replication slave on *.* to 'wyt'@'%' identified by '123456';
```
4.slave添加授权用户的信息

```bash
#先查看master当前二进制文件状态
mysql>show master status;
+----------------------+----------+--------------+------------------+-------------------+
| File                 | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+----------------------+----------+--------------+------------------+-------------------+
| localhost-bin.000006 |      436 |              |                  |                   |
+----------------------+----------+--------------+------------------+-------------------+

#到从服务器进入MySQL
mysql>stop slave #停止slave
CHANGE MASTER TO MASTER_HOST='192.168.119.191',
MASTER_USER='wyt',
MASTER_PASSWORD='123456',
MASTER_LOG_FILE='localhost-bin.000006',
MASTER_LOG_POS=436;
#查看slave状态
mysql>show slave status\G;
#可以看到 Slave_IO_Running: No  Slave_SQL_Running: No 所以需要开启
mysql>start slave;#开启slave
```
可以看到
 Slave_IO_Running: YES
  Slave_SQL_Running: YES
![在这里插入图片描述](https://img-blog.csdnimg.cn/6c6c7b75a52247858d7f3c0526ff97ca.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

检验：我们在主服务器上建库，会同步到从服务器。

## 延迟备份

延时备份让slave滞后于master一段时间,当你误操作时只要立即停止slave的同步,即可轻松地从延时备份库中找回你误删的数据，可以起到全备的作用。
```bash
mysql>stop slave #停止slave
mysql>CHANGE MASTER TO MASTER_DELAY = 3600; 延迟3600秒
mysql>start slave;
```


## xtrabackup
因为意外导致某个MySQL的从服务器宕机，且不可修复，因为是业务数据库，不能停机和锁表进行从库的搭建，二进制日志文件和位置号发生变化不好确定，所以考虑使用xtrabackup工具进行在线主从搭建。

## 主从切换
主从复制其中一个目的就是，当主服务器出现问题时，可以主从切换，切换到从服务器，提高并发性能，实现高可用，那么**什么时候主从切换，如何手工实现主从切换**

> 当主服务器宕机，且不可修复，需要提升从服务器为主服务器。
> 
 ### 主从切换手工步骤
 
主从切换的步骤也就是把从服务器切换为主服务器，然后再配置一台从服务器。
- stop slave
- reset  master
- 开启二进制日志
- 建立授权复制的用户
- 再启动一台服务器做从，配置master信息去拉取二进制日志

### 主从切换脚本步骤
- 监控master，在另外一台机器扫描端口 `nc 3306`
- 直接访问 `mysql -uroot -p123456 -h 192.168.119.191 -e show databases;`
- 每秒钟监控一次
