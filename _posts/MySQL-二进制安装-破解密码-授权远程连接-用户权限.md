## 二进制脚本一键安装
```bash
wget https://cdn.mysql.com/archives/mysql-5.7/mysql-5.7.34-linux-glibc2.12-x86_64.tar.gz #下载二进制安装包
bash <(curl -s -L https://cdn.jsdelivr.net/gh/yutao517/blog@main/bash/one-key-install-mysql.sh)
su -root #切换到root用户加载
service mysqld start
ps aux|grep mysqld #判断MySQL是否运行
```
**mysqld和mysql_safe**

mysql_safe是父进程，mysqld是子进程

## 允许远程连接MySQL
		
第一种方法
```bash
mysql>grant all on *.* to 'root'@'%' identified by '123456';
#grant是授权命令，all表示所有的权限，on *.*在所有库里的所有表，第一个*表示库，第二个*表示表，to 'root'@'%'表示允许root这个用户从任何地方连接过来登录，设置密码为123456
```
第二种方法
```bash
mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
mysql>flush privileges;#设置完成后打开服务器安全组配置放行端口3306
```
## 修改MySQL密码为123456
```bash
mysql>alter user 'root'@'localhost' identified by '123456';
#set password for root@localhost = password('123456'); 
```
## MySQL重置密码为123456

```bash
bash <(curl -s -L https://cdn.jsdelivr.net/gh/yutao517/blog@main/bash/reset-mysql-pwd.sh)
mysql -uroot -p123456 #登录
```
## 创建新用户授权权限

```bash
mysql>create user 'wyt'@'localhost' identified by '123456';
mysql>select user,host from mysql.user; #查看用户
+---------------+-----------+
| user          | host      |
+---------------+-----------+
| root          | %         |
| mysql.session | localhost |
| mysql.sys     | localhost |
| root          | localhost |
| wyt           | localhost |
+---------------+-----------+

mysql>grant select on wyt.* to 'wyt'@'%'; #授权wyt用户从任意地方登录，只能查看wyt数据库
mysql>grant all on *.* to 'wyt'@'%' with grant option; #授权wyt用户所有权限操作所有库表，并且还可以给其他用户授权。
mysql>show grants for wyt; #查看wyt用户的权限
+------------------------------------------------------------+
| Grants for wyt@%                                           |
+------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'wyt'@'%' WITH GRANT OPTION |
| GRANT SELECT ON `wyt`.* TO 'wyt'@'%'                       |
+------------------------------------------------------------+
mysql>revoke all on *.* from 'wyt'@'%'; #废除权限
mysql>drop user 'wyt'@'localhost';
```


## 查看MySQL版本

```bash
1.登录的时候版本提示
2. mysql>select version();
+-----------+
| version() |
+-----------+
| 5.7.34    |
+-----------+
3.mysql>show variables like 'version';
+---------------+--------+
| Variable_name | Value  |
+---------------+--------+
| version       | 5.7.34 |
+---------------+--------+

```

## Windows无法远程连接到MySQL的原因

> 1.windows网络问题
> 
> 2.是否授权用户远程登陆
> 
> 3.linux防火墙问题
> 
> 4.检查mysqld服务是否打开
> 
> 5.检查下端口号是否修改
> 
> 6.云服务器安全组配置

## 基本命令

```bash
mysql>show variables\G; #查看变量\G以行显示
mysql>show character set; #查看字符集
mysql>show processlist; #查看登录mysql的用户
mysql>select user(); #查看当前登录mysql的用户
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
mysql>select database();#查看当前所在的库
+------------+
| database() |
+------------+
| wyt        |
+------------+
```
## 默认的四个库

```bash
mysql>show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```
- **information_schema**
信息数据库，保存着关于MySQL服务器所维护的所有其他数据库的信息。如数据库名，数据库的表，表栏的数据类型与访问权限等。普通用户默认只能看到这个库。
- **performance_schema** 
性能架构库，用于收集数据库服务器性能参数。
- **sys**
把performance_schema的把复杂度降低，让DBA能更好的阅读这个库里的内容。让DBA更快的了解DB的运行情况。
- **mysql**
负责存储数据库的用户、权限设置、关键字等mysql自己需要使用的控制和管理信息，行使管理功能。

