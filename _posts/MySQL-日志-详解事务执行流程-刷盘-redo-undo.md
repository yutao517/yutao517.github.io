日志是用来排错，做数据分析，了解程序运行情况的。
# 错误日志
如果不知道错误日志的位置，可以查看变量。
```bash
mysql>show variables like '%log%';
#模糊查询log的位置，里面有log_error
mysql>show variables like 'log_error'; #精准查询错误日志位置
+---------------+-----------------------------+
| Variable_name | Value                       |
+---------------+-----------------------------+
| log_error     | ./localhost.localdomain.err |
+---------------+-----------------------------+
```
# 慢查询日志
慢查询日志：记录在 MySQL 中响应时间超过阀值的语句，指运行时间超过long_query_time值的 SQL。

```bash
mysql>show variables like '%long_query%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
#默认时长为10s,所以超过10ms的sql语句会被记录。
```
**比如数据库负载压力特别高解决思路**

> 1.从慢日志中分析sql语句响应时长比较长的语句，进行优化。
> 2.业务量大，硬件达到极限

```bash
 mysql>show variables like 'slow_query%';
+---------------------+--------------------------------+
| Variable_name       | Value                          |
+---------------------+--------------------------------+
| slow_query_log      | OFF                            |
| slow_query_log_file | /data/mysql/localhost-slow.log |
+---------------------+--------------------------------+
#默认慢查询日志关闭
vim /etc/my.cnf 
# mysqld增加两行行
#slow_query_log=1 
#long_query_time = 1

>[mysqld]
>socket=/data/mysql/mysql.sock
>port = 3306
>open_files_limit = 8192
>innodb_buffer_pool_size = 512M
>character-set-server=utf8
#skip-grant-tables
>slow_query_log=1
>long_query_time = 1
#:wq退出并保存
service mysqld restart #重启
mysql>show variables like '%slow_query%';
+---------------------+--------------------------------+
| Variable_name       | Value                          |
+---------------------+--------------------------------+
| slow_query_log      | ON                             |
| slow_query_log_file | /data/mysql/localhost-slow.log |
+---------------------+--------------------------------+
#on已经打开
mysql>show variables like '%long_query%';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 1.000000 |
+-----------------+----------+
#设置时长为1s
mysql>select sleep(2); #测试命令休息两秒
mysql>cat /data/mysql/localhost-slow.log #该命令超过1s，查看日志中有没有该命令。
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/cf133321e21e4eddbfdecb5ee429bcbe.png)

> 官方文档的意思是该值可以精确到毫秒，不是值默认值的单位是毫秒，实测改值是秒，因为select
> sleep(0.5)这条命令无法写入慢日志，因为他是0.5秒，低于1秒

# 通用日志

```bash
 mysql>show variables like '%general%';
+------------------+---------------------------+
| Variable_name    | Value                     |
+------------------+---------------------------+
| general_log      | OFF                       |
| general_log_file | /data/mysql/localhost.log |
+------------------+---------------------------+
#配置文件[mysqld]新增一行general_log打开
vim /etc/my.cnf 
>[mysqld]
>socket=/data/mysql/mysql.sock
>port = 3306
>open_files_limit = 8192
>innodb_buffer_pool_size = 512M
>character-set-server=utf8
>general_log
#配置文件
mysql>show variables like '%general%';
+------------------+---------------------------+
| Variable_name    | Value                     |
+------------------+---------------------------+
| general_log      | ON                        |
| general_log_file | /data/mysql/localhost.log |
+------------------+---------------------------+
#实际生产中通用日志不介意打开，因为操作命令太多会占用磁盘空间，所以日志存放空间要考虑空间。
mysql>set global general_log = 1;#所以选择临时打开
```

# 二进制日志
**二进制日志**：记录修改数据或有可能引起数据变更的sql语句，这是一个二进制日志。

**作用**：恢复数据和主从复制

```bash
#配置文件[mysqld]新增两行打开
log_bin
server_id = 1 #区别服务器ID，主从复制会使用。
service mysqld restart #重启mysql服务
mysql>show variables like 'log_bin%';
+---------------------------------+---------------------------------+
| Variable_name                   | Value                           |
+---------------------------------+---------------------------------+
| log_bin                         | ON                              |
| log_bin_basename                | /data/mysql/localhost-bin       |
| log_bin_index                   | /data/mysql/localhost-bin.index |
| log_bin_trust_function_creators | OFF                             |
| log_bin_use_v1_row_events       | OFF                             |
+---------------------------------+---------------------------------+

 mysql>show master status; #查看服务器当前使用的二进制日志文件的状态信息
+----------------------+----------+--------------+------------------+-------------------+
| File                 | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+----------------------+----------+--------------+------------------+-------------------+
| localhost-bin.000001 |      154 |              |                  |                   |
+----------------------+----------+--------------+------------------+-------------------+
mysql>show variables like 'max_binlog_size'; #二进制日志文件最大大小1G
+-----------------+------------+
| Variable_name   | Value      |
+-----------------+------------+
| max_binlog_size | 1073741824 |
+-----------------+------------+

mysql>select 1073741824/1024/1024;
+----------------------+
| 1073741824/1024/1024 |
+----------------------+
|        1024.00000000 |
+----------------------+
mysql>flush logs; #刷新二进制日志
mysql>show binary logs;#查看所有二进制文件和大小
+----------------------+-----------+
| Log_name             | File_size |
+----------------------+-----------+
| localhost-bin.000001 |       205 |
| localhost-bin.000002 |       154 |
+----------------------+-----------+

mysqlbinlog localhost-bin.000001 #读取二进制日志，不能用cat读取

mysql>reset master; #重置所有二进制日志文件

```
一个二进制日志文件记录了整个msql进程里所有的库的操作，都会记录到一个二进制文件里，如果需要记录到不同的日志文件里可以采用**多实例**。

**多实例**：就是多个进程，每个进程监听的端口不同(3306,3307)，有多个配置文件。MySQL中一个mysqld就是一个实例。

**二进制日志格式**

```bash
 mysql>show variables like 'binlog_format';#查看默认二进制格式
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+

```
- Statement：记录用户输入的SQL语句，不记录每一行数据的变化，减少了 bin-log 日志量，修改数据的时候使用了某些函数的时候会出现问题，比如：now() 函数在有些版本中就不能被正确复制。
- Row：记录表里行数据发生的变化，记录的是数据行的更改情况，即数据行在更改前、更改后的变化情况，比如使用now函数主从复制可以使用主服务器的时间，而statement做不到，在主从复制模式下可靠性最好，但是更消耗磁盘空间。
- Mixed：前两种模式的混合，根据实际情况判断

**删除二进制文件**

```bash
mysql>reset master; #重置所有二进制日志文件
mysql>purge binary logs before '2022-04-06:11:00:00'; #删除日期之前的
mysql>purge binary logs to 'localhost-bin.000002';#将指定日志文件之前的
mysql>set global expire_logs_days = 3; #临时设置三天后清理，不推荐，服务重启会导致失效。所有推荐写到配置,[mysqld]新增一行 explore_logs_days = 3
mysql>show variables like 'expire_logs_days'; #查看二进制日志保存时长
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| expire_logs_days | 3     |
+------------------+-------+
```
## 事务完整执行过程
了解刷盘前先看一下事务执行过程 
例如：`mysql>update test set wyt = 'yutao' where id=1;`更新一行数据
1. .事务开始
2. 从磁盘中读取id=1这行数据到内存缓冲池(Buffer Pool)
3. 申请锁资源，对这行数据上排他锁
4. 将需要修改的数据页读取到缓冲池
5. 记录修改前的这行数据到undo log(一个刷盘过程）
6. 修改数据(数据直接写入磁盘，没有刷盘过程)
7. 修改后的数据写入到 redo log buffer(一个刷盘过程)
8. 第6步修改后的数据准备提交
9. binlog二进制日志文件写入磁盘提交事务，成功，失败就回滚或者重做
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ccec46f05b94ef6a5e6d5db3916d101.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)



![在这里插入图片描述](https://img-blog.csdnimg.cn/b3bd4204f9e044d8954bfefa50e710e1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)
## 刷盘
**刷盘**：指的是MySQL先从内存中的Buffer Pool(缓冲池)中将内容写到内存OS Buffer(操作系统缓冲)中，再**fsync**从内存的OS Buffer数据永久写到系统磁盘上。

**刷盘时机参数**

- sync_binlog=[N]：表示写缓冲多少次，刷一次盘。默认值为0。取值是0，1，N三种值。
- binlog_cache_size：二进制日志缓冲部分的大小，默认值为32k，设置过大会造成内存浪费。设置过小，会频繁将缓冲日志写入临时文件。
- sync_binlog=0：表示刷新binlog时间点由操作系统自身来决定，操作系统自身会每隔一段时间就刷新缓冲数据到磁盘，能减少刷盘次数，这个性能好。
- sync_binlog=1：表示每次事务提交都要调用fsync()，刷新binlog写入到磁盘。这个可靠性高，默认为1
	```bash
	mysql>show variables like 'sync_binlog';
	+---------------+-------+
	| Variable_name | Value |
	+---------------+-------+
	| sync_binlog   | 1     |
	+---------------+-------+
	```
- sync_log=N：表示N个事务提交，才会调用fsync()进行一次binlog刷新，写入磁盘。

## redolog和undolog

**重做日志（redo log）**：确保事务的持久性， 防止在发生故障的时间点，尚有脏页未写入磁盘(比如正在写数据突然停电)，在重启mysql服务的时候，根据redo log进行重新做一次，从而达到事务的持久性这一特性。

**回滚日志（undo log）**：保存了事务发生之前的数据的一个版本，提供回滚和多个⾏版本控制。如果因为某些原因导致事务失败，ROLLBACK可以借助undo log进⾏回滚。
