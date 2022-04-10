## 备份

### 运行状态分类

**热备份**：在数据库运行时，直接进行备份，对运行的数据库没有影响

**冷备份**：在数据库停止运行的时候进行备份，这种备份方式最为简单，只需要拷贝数据库物理文件

温备：在数据库运行的时候进行备份的，但对当前数据库的操作会产生影响。

### 备份方式分类

**物理备份**：直接复制数据文件对磁盘进行拷贝（备份整个磁盘文件）

**逻辑备份**：从数据库中导出数据另存而进行的备份（逻辑备份的是工艺过程，重做一次）

### 业务分类
![在这里插入图片描述](https://img-blog.csdnimg.cn/b477b439330b458f89d256dbaaba93e7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_17,color_FFFFFF,t_70,g_se,x_16)

**完全备份**：对整个数据库的数据和数据结构进行备份

> 如果在星期五数据被破坏了，则只需要还原星期五完全的备份，结合二进制日志备份

 
 **差异备份**：备份自上一次**完全备份**之后有变化的数据

>  如果在星期五数据被破坏了，则只需要还原星期一完全的备份和星期四的差异备份，结合二进制日志备份。

 
  **增量备份**：备份自上一次备份（包含完全备份、差异备份、增量备份）之后有变化的数据

>   如果在星期五数据被破坏了，则你需要还原星期一正常的备份和从星期二至星期五的所有增量备份和二进制日志备份，还原耗时。

## 还原
### mysqldump的备份使用
```bash
mysqldump -uroot -p123456 --all databases >dump.sql #备份所有库
mysqldump -uroot -p123456 --databases db1 db2 db3 >dump.sql #备份db1、db2、db3库
mysqldump -uroot -p123456 test t1 t2 t3 >dump.sql #备份test库里的t1、t2、t3表
```

### mysql的还原使用

```bash
mysql </backup/wyt.sql; #恢复备份的库但不包括插入的数据，插入的数据要用二进制日志恢复。
```
### 二进制日志还原数据

```bash
mysqlbinlog -v localhost-bin.000002|egrep -C20 'drop database wyt' #查看删库前后20行操作，主要看前,egrep -B20,找结束点
mysqlbinlog -v localhost-bin.000002|more
#以分页显示方便找起始点
mysqlbinlog   --start-datetime="2022-04-07 16:21:15" --stop-datetime="2022-04-27 16:35:14" /data/mysql/localhost-bin.000002| mysql -uroot -p123456 #时间点恢复
mysqlbinlog --start-position=1006 --stop-position=1868 /data/mysql/localhost-bin.000002| mysql -u root -p123456 #位置点恢复
```

## 备份还原整个流程
```bash
mysql>reset master; #因为数据这里不重要所以从头开始直接重置二进制文件
mysql>flush logs;#产生一个新的二进制文件
mysql>show master status; #查看当前使用的二进制文件是localhost-bin.000002 
mkdir -p /backup
mysqldump -uroot -p123456 --databases wyt >/backup/wyt.sql #备份wyt库，存到/backup目录下名字为wyt.sql
mysql>use wyt; #使用wyt库
mysql>insert into wyt(id) values(1); #在wyt库下的wyt表插入一条数据id=1
mysql>drop database wyt; #删wyt库
mysql </backup/wyt.sql; #恢复备份的库但不包括插入的数据，插入的数据要用二进制日志恢复。
cd /data/mysql #进入mysql二进制文件目录
mysqlbinlog -v localhost-bin.000002|egrep -C20 'drop database wyt' #查看删库前后20行操作，主要看前,egrep -B20,找结束点
mysqlbinlog -v localhost-bin.000002|more
#以分页显示方便找起始点
mysqlbinlog   --start-datetime="2022-04-07 16:21:15" --stop-datetime="2022-04-27 16:35:14" /data/mysql/localhost-bin.000002| mysql -uroot -p123456 #时间点恢复
mysqlbinlog --start-position=1006 --stop-position=1868 /data/mysql/localhost-bin.000002| mysql -u root -p123456 #位置点恢复
```
[官方参考文档](https://dev.mysql.com/doc/refman/5.7/en/point-in-time-recovery-positions.html)
