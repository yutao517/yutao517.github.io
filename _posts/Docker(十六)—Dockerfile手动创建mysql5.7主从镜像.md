---
layout: article
title: Docker(十六)—Dockerfile手动创建mysql5.7主从镜像
tags: Docker
category: blog
date: 2022-06-24 17:28:43 +08:00
mermaid: true
---
参考链接
[https://zhuanlan.zhihu.com/p/122578441](https://zhuanlan.zhihu.com/p/122578441)
## mysql-master
**1、首先创建Dckerfile**

Dockerfile 长这样，具体的命令上面有解释。核心逻辑就是拷贝两个文件进去，然后在容器启动的时候执行 conf.sh，由 conf.sh 执行另外一个文件。
```bash
From mysql:5.7.25
MAINTAINER 1271030564@qq.com
#设置免密登录 
ENV MYSQL_ALLOW_EMPTY_PASSWORD yes
#将所需文件放到容器中
COPY conf.sh /mysql/conf.sh
COPY privileges.sql /mysql/privileges.sql
#设置容器启动时执行的命令
CMD ["sh", "/mysql/conf.sh"]
```
首先设置允许免密登录是为了方便后面配置，密码最后再通过 privileges.sql 来设置。

**2、编写容器启动脚本conf.sh**

```bash
#!/bin/bash 
set -e
#查看mysql服务的状态，方便调试，这条语句可以删除
echo '1. set server_id....'

sed -i '/\[mysqld\]/a server-id=1\nlog-bin=/var/log/mysql/mysql-bin\ngtid-mode=ON\nenforce-gtid-consistency=ON' /etc/mysql/mysql.conf.d/mysqld.cnf
 
echo '2. start mysql...'
service mysql start
 
echo '3. setting password...'
sed -i 's/MYSQLROOTPASSWORD/'$MYSQL_ROOT_PASSWORD'/' /mysql/privileges.sql
sed -i 's/MYSQLREPLICATIONUSER/'$MYSQL_REPLICATION_USER'/' /mysql/privileges.sql
sed -i 's/MYSQLREPLICATIONPASSWORD/'$MYSQL_REPLICATION_PASSWORD'/' /mysql/privileges.sql
mysql < /mysql/privileges.sql
 
echo '4. service mysql status'
echo 'mysql for intsig if ready...'
 
tail -f /dev/null
```
**3、privileges.sql**

这个文件是对 MySQL 进行一些权限配置，比如设置用户密码，创建新用户，数据库授权等。

```bash
use mysql;
set password for root@'localhost' = password("MYSQLROOTPASSWORD");
grant all on *.* to "MYSQLREPLICATIONUSER"@'%' identified by "MYSQLREPLICATIONPASSWORD" with grant option;
flush privileges;
```
**4、制作镜像上传至镜像仓库**

![在这里插入图片描述](https://img-blog.csdnimg.cn/88bf27f835694682a898d02fb7663073.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/859e1cb261e74584b514ddb727488f53.png)

**启动容器测试配置是否成功**
```bash
docker run  --name yutaosql -e MYSQL_ROOT_PASSWORD="1234567" -e MYSQL_REPLICATION_USER="repl"  -e MYSQL_REPLICATION_PASSWORD="1234567"  -d yutao517/mysql5.7-master:2.1
```
**链接容器测试登陆**
```bash
docker exec -it yutaosql bash
mysql -uroot -p1234567
```
**查看权限**
```bash
mysql> select * from mysql.user where user='repl'\G
```
![](https://img-blog.csdnimg.cn/4d7e3e7c71ca49448f5b611f5d8abbd6.png)

成功创建mysql-master镜像

## mysql-slave
**Dockerfile**

```bash
From mysql:5.7.25
MAINTAINER 1271030564@qq.com

ENV MYSQL_ALLOW_EMPTY_PASSWORD yes
COPY conf.sh /mysql/conf.sh
COPY privileges.sql /mysql/privileges.sql

CMD ["sh", "/mysql/conf.sh"]
```
**启动脚本**

```bash
#!/bin/bash

set -e

echo '1. set server_id...'
RAND="$(date +%s | rev | cut -c 1-2)$(echo ${RANDOM})"
sed -i '/\[mysqld\]/a server-id='$RAND'\nlog-bin=/var/log/mysql/mysql-bin\ngtid-mode=ON\nenforce-gtid-consistency=ON' /etc/mysql/mysql.conf.d/mysqld.cnf


echo '2. start mysql...'
service mysql start

echo '3. setting password...'
sed -i 's/MYSQLROOTPASSWORD/'$MYSQL_ROOT_PASSWORD'/' /mysql/privileges.sql
sed -i 's/MYSQLMASTERSERVICEHOST/'$MYSQL_MASTER_SERVICE_HOST'/' /mysql/privileges.sql
sed -i 's/MYSQLREPLICATIONUSER/'$MYSQL_REPLICATION_USER'/' /mysql/privileges.sql
sed -i 's/MYSQLREPLICATIONPASSWORD/'$MYSQL_REPLICATION_PASSWORD'/' /mysql/privileges.sql


mysql < /mysql/privileges.sql

echo '4. service mysql status'
echo 'mysql for tigerfive if ready...'

tail -f /dev/null
```
**privileges.sql**

```bash
use mysql;
set password for root@'localhost' = password('MYSQLROOTPASSWORD');
flush privileges;
CHANGE MASTER TO master_host='MYSQLMASTERSERVICEHOST', master_user='MYSQLREPLICATIONUSER', master_password='MYSQLREPLICATIONPASSWORD' ;
START SLAVE;
```
**制作镜像上传至镜像仓库**
```bash
docker build -t yutao517/mysql5.7-slave:2.1  .
```

```bash
docker push yutao517/mysql5.7-slave:2.1
```

**查看mysql-master的容器IP**
```bash
docker inspect yutaosql |grep IPAddr
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2efd43eb84db477eaeef49288a118ced.png)
**启动命令**

```bash
docker run  --name yutaosql2 -e MYSQL_ROOT_PASSWORD="1234567" -e MYSQL_MASTER_SERVICE_HOST="172.17.0.3" -e MYSQL_REPLICATION_USER='repl' -e MYSQL_REPLICATION_PASSWORD="1234567"   -d yutao517/mysql5.7-slave:2.1
```
## 验证
**mysql-master插入数据**

```bash
docker exec -it yutaosql bash  
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/36fab310fcf2478dac5de282febb228e.png)

**mysql-slave查看是否同步**


![在这里插入图片描述](https://img-blog.csdnimg.cn/a302f4d554934777b3d342ee123ac79f.png)


