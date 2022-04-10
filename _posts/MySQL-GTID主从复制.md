## 概念
**全局事务标识符（GTID)** 的全称为Global Transaction Identifier，由server_uuid和事务id组成，是在整个复制环境中对一个事务的唯一标识。
## 作用
主从故障切换中，比起bin_log+pos的传统方式，因为每个事物都对应唯一GTID，不需要指定二进制文件名和位置减少手工干预和降低服务故障时间，以前做过的不再执行，可以节约时间。

## 步骤
环境：成功搭建基本异步配置的前提。
主服务器IP地址：192.168.2.198 

**master配置**
```bash
mysql>grant replication slave on *.* to 'wyt'@'%' identified by '123456'; #授权一个复制用户wyt密码是123456
```

```bash
vim /etc/my.cnf 
```
mysqld需要添加以下配置
```bash
[mysqld]
log_bin 
server_id = 1
gtid-mode = on #打开gtid功能
enforce-gtid-consistency = on  #打开gtid一致性
```
**slave配置**

```bash
log_bin 
server_id = 2
gtid-mode = on #打开gtid功能
enforce-gtid-consistency = on  #打开gtid一致性
log_slave_updates = on # 将来着master的更新记录到自己的二进制日志中。
```

```bash
CHANGE MASTER TO MASTER_HOST='192.168.2.198',
MASTER_USER='wyt',
MASTER_PASSWORD='123456',
MASTER_auto_position=1;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/330e539f1d0145a795ab5e150f08a25c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
