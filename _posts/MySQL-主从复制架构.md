## 一主多从复制架构

在主库读取请求压力非常大的场景下，可以通过配置一主多从复制架构实现读写分离，把大量的对实时性要求不是特别高的读请求通过负载均衡分布到多个从库上，对于实时性要求很高的读请求可以让从主库去读，降低主库的读取压力。

### 一主多从再级联（多级复制）

其他配置操作没有区别，但是**log_slave_updates**参数的状态必须设置ON，从库二进制日志log_bin也要打开

mysql的官网说明如下：

> Normally, a slave does not log to its own binary log any updates that
> are received from a master server. This option tells the slave to log
> the updates performed by its SQL thread to its own binary log. For
> this option to have any effect, the slave must also be started with
> the --log-bin option to enable binary logging

> 通常，从站不会将任何更新记录到自己的二进制日志中 从主服务器接收。此选项告诉从站记录 其 SQL 线程对其自己的二进制日志执行的更新。为了这个选项有任何效果，slave也必须启动。

## 双主复制架构

适用于DBA做维护时需要主从切换的场景，通过双主复制架构避免了重复搭建主从库的麻烦，主库Master1和Master互为主从。


> 在做数据库的主主同步时需要设置自增长的两个相关配置：auto_increment_offset和auto_increment_increment。
> auto-increment-increment表示自增长字段每次递增的量，其默认值是1。它的值应设为整个结构中服务器的总数，我这里用到两台服务器，所以值设为2。
> auto-increment-offset是用来设定数据库中自动增长的起点(即初始值)，因为这两能服务器都设定了一次自动增长值2，所以它们的起点必须得不同，这样才能避免两台服务器数据同步时出现主键冲突。

