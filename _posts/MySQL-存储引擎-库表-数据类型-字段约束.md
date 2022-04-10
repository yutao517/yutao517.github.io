## SQL(结构化查询语言）
**SQL**：数据库查询和程序设计语言

**语句分类**
 |  类型  | 说明|
 |  ----  | ----  | 
 | 数据定义语言 | DDL(Data Definition Language),定义数据库对象,create,drop,alter
 |数据操作语言| DML(Data Manipulation Language),操作数据库中的数据,insert,delete,update
 |数据查询语言|DQL(Data Query Language),)查询数据,select
 |数据控制语言|DCL(Data Control Language),控制数据库对象的权限,greate,revoke
## 建库建表
**库操作**
```bash
mysql>create database wyt; #建库
mysql>show databases; #查看数据库
mysql>show create database wyt; #查看这个库的字符集类型
mysql>show variables like "%character%"; #查看包含字符集的变量
mysql>use wyt; #进入wyt这个数据库
mysql>drop database wyt; #删除wyt这个数据库
```
**表操作**

```bash
mysql>use mysql; #进入mysql数据库
mysql>create table wyt.t1(id int,name varchar(20)); #使用mysql库，在wyt库中创建t1表，如果就在wyt库，wyt.可以省略
mysql>show tables from wyt; #查看wyt库中的表，如果就在wyt库,from wyt可以省略
mysql>show tables in wyt; #查看wyt库中的表，如果就在wyt库,in wyt可以省略
mysql>create table if not exists wyt.t1(id int,name varchar(20)); #如果存在该表不报错
mysql>show warnings; #查看警告
mysql>create table t2(id int(4) not null primary key,name varchar(10) not null); #非空，primary key主键唯一性。
mysql>desc t2; #查看表结构
mysql>show create table t2; #查看t2表详情
mysql>insert into t2(id,name) values(1,"王宇涛");
mysql>select * from wyt.t2 #查表
mysql>delete from t2 where id = 1; #删表
mysql>truncate t3; 
```
**update更新**

```bash
mysql>update t2 set name = 'wyt' where id =1;
mysql>update t2 set name = replace(name,'wyt','王宇涛') where id =1;
mysql>select * from t7 ORDER BY score ASC; #成绩升序，DESC降序
```

**TRUNCATE和DELETE的区别**

> truncate 适合删除大表，速度非常快，理解为直接清空表空间，不会产生二进制日志，不能通过日志去恢复，只能通过原来的备份去恢复。
> delect 删除数据非常慢，但会产生二进制日志，可以恢复。

## 数据库类型
**关系型数据库**：采用二维表格模型来组织数据的数据库。

> MySQL，Oracle，Microsoft SQL Server，MSSQL，postgreSQL

**非关系型数据库**：非关系型的，分布式的，结构不固定，不规则。

> redis，mongoDB，TIDB，timeseriesDB

**结构化数据**：可以使用关系型数据库表示和存储，可以用二维表结构来逻辑表达实现的数据。

**非结构化数据**：结构化数据之外的一切数据，就是字段可变的的数据，可以是文本也可以是非文本。

**半结构化数据**：结构化数据的一种形式，不符合关系型数据库结构模型，但包含相关标记。例如html，json，xml。

## 数据类型
数值类型、字符串（字符和字节）类型、日期和时间类型、空间类型和 JSON数据类型

**数值类型**

1. 整数类型
	
	|  类型  |存储需求(bytes)|带符号的范围|无符号的范围|
	|  ----  | ----  | ---- | ----|
	|tinyint|1字节|-128~127|0~2^8-1
	|smallint|2字节|-2^15 ~ 2^15-1|0~2^16-1
	|mediumint|3字节|-2^23 ~ 2^23-1|0~2^24-1
	|int|4字节|-2^31 ~ 2^31-1|0~2^32-1
	|bigint|8字节|-2^63 ~ 2^63-1|0~2^64-1

2. 定点类型
DECIMAL、NUMERIC

	```bash
	mysql>create table wyt.t3(name varchar(20),salary decimal(5,2));
	```
	标准 SQL 要求DECIMAL(5,2)能够存储任何 5 位和 2 位小数的值，因此salary 列中可以存储的值范围从-999.99到 999.99.

3. 浮点类型
FLOAT、DOUBLE
4. 位值类型
BIT

**字符串类型**

1. CHAR 和 VARCHAR 
**CHAR**是固定长度的字符串，长度可以是 0 到 255 之间的任何值。
**VARCHAR**是可变长度的字符串，长度可以指定为 0 到 65535 之间的值，另外会多存储一个字节。

	| 值  |CHAR(4)|存储需求(bytes)|VARCHAR(4)|存储需求(bytes)|
	|  ----- | ----  | ---- | ----| ---|
	|‘ ’ |'     '|4字节|‘’|1字节
	|‘ab’|'ab'|4字节|‘ab’|3字节
	|'abcd'|'abcd'|4字节|'abcd'|5字节
	|'abcdefg'|'abcd'|4字节|'abcd'|5字节
2. BLOB和TEXT
BLOB是二进制对象，分为TINYBLOB、BLOB、 MEDIUMBLOB和LONGBLOB。
TEXT分为TINYTEXT、TEXT、 MEDIUMTEXT和LONGTEXT。
3. ENUM(枚举) 类型
只能在创建表时所规范中明确枚举的允许值列表，比如性别，只能说男或者女

	```bash
	CREATE TABLE shirts (
	    name VARCHAR(40),
	    size ENUM('x-small', 'small', 'medium', 'large', 'x-large')
	);
	INSERT INTO shirts (name, size) VALUES ('dress shirt','large'), ('t-shirt','medium'),
	  ('polo shirt','small');
	SELECT name, size FROM shirts WHERE size = 'medium';
	+---------+--------+
	| name    | size   |
	+---------+--------+
	| t-shirt | medium |
	+---------+--------+
	UPDATE shirts SET size = 'small' WHERE size = 'large';
	COMMIT;
	```
4. SET 类型
集合，不允许出现重复值
5. BINARY 和 VARBINARY类型
类似CHAR VARCHAR，但是存放二进制字符串

**日期和时间类型**

DATE、 TIME、 DATETIME、 TIMESTAMP 、YEAR

![在这里插入图片描述](https://img-blog.csdnimg.cn/0630599c9eeb4e6282d2b4c2327f2a6d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeXV0YW9fNTE3,size_20,color_FFFFFF,t_70,g_se,x_16)

## 字段属性

auto_increment ：自增。
not null ：非空。 
unique ：唯一,空值也只能出现一次。 
primary key ：主键 相当于not null+unique，非空且唯一。 
default ：设定默认值 comment 注释
comment ：注释描述

```bash
mysql>create table t4(id int auto_increment not null primary key comment '编号',name varchar(20) default '王宇涛');
mysql>desc t4;
+-------+-------------+------+-----+-----------+----------------+
| Field | Type        | Null | Key | Default   | Extra          |
+-------+-------------+------+-----+-----------+----------------+
| id    | int(11)     | NO   | PRI | NULL      | auto_increment |
| name  | varchar(20) | YES  |     | 王宇涛    |                |
+-------+-------------+------+-----+-----------+----------------+

mysql>insert into t4() values(); #插入的可以不用写，id默认为1，自增，name默认为王宇涛
mysql>select * from t4;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | 王宇涛    |
+----+-----------+
```
外键FOREIGN KEY
```bash
#创建父表
mysql>create table dep(depid int not null primary key,dname varchar(20));
mysql>insert into dep(depid,dname) values(10,'市场部');
mysql>insert into dep(depid,dname) values(20,'营销部');
#创建子表
mysql>create table emp(id int primary key,name varchar(20),depid int, foreign key(depid) references dep(depid));
mysql>insert into emp(id,name,depid) values(1,'张三','10');
mysql>insert into emp(id,name,depid) values(2,'李四','20');
mysql>insert into emp(id,name,depid) values(3,'王五','30');
#ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`wyt`.`emp`, CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`depid`) REFERENCES `dep` (`depid`))
#第三条王五报错,因为被dep表约束，dep表没有30这个depid
mysql>select * from dep;
+-------+-----------+
| depid | dname     |
+-------+-----------+
|    10 | 市场部    |
|    20 | 营销部    |
+-------+-----------+
mysql>select * from emp;
+----+--------+-------+
| id | name   | depid |
+----+--------+-------+
|  1 | 张三   |    10 |
|  2 | 李四   |    20 |
+----+--------+-------+
#30王五信息并没有写进去
```
外键可以避免冗余，但数据库数据太大避免使用，因为需要关联数据库，消耗CPU和内存
## NULL值和空值的区别
- 空值('')的长度是0，不占用空间的，NULL长度是NULL，占用空间
- 查NULL值列，则使用 is NULL去查，查空值('')列，则使用 =''
- 进行count()统计某列的记录数的时候，如果采用的NULL值，会别系统自动忽略掉，但是空值是会进行统计到其中的。
- 如果没有特殊的业务场景，可以直接使用空值。

## 存储引擎
**存储引擎**：将MySQL在内存中存储的所有表格数据写到磁盘中，将磁盘中的数据读取出来。

	
```bash
mysql>show engines; #查看存储引擎	
mysql>create table wyt(id int) engine=myisam default charset=utf8mb4; #指定存储引擎和默认字符集
mysql>show create table wyt; #查看wyt表信息
+-------+------------------------------------------------------------------------------------------+
| Table | Create Table                                                                             |
+-------+------------------------------------------------------------------------------------------+
| wyt   | CREATE TABLE `wyt` (
  `id` int(11) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 |
+-------+----------------------------------------
mysql>show table status from wyt where name='wyt'\G; #查看表信息
mysql>show table status like 'wyt'\G; #查看表信息
*************************** 1. row ***************************
           Name: wyt
         Engine: MyISAM
        Version: 10
     Row_format: Fixed
           Rows: 0
 Avg_row_length: 0
    Data_length: 0
Max_data_length: 1970324836974591
   Index_length: 1024
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2022-03-31 21:46:46
    Update_time: 2022-03-31 21:46:46
     Check_time: NULL
      Collation: utf8mb4_general_ci
       Checksum: NULL
 Create_options: 
        Comment: 
```

- **InnoDB**：InnoDB是事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定，外键，InnoDB是默认的MySQL引擎。
- CSV：由逗号分割数据的存储引擎。
- MyISAM： 拥有较高的插入，查询速度，但不支持事务。
- BlackHole ：黑洞引擎，写入的任何数据都会消失。
- Memory ：临时存放数据，内容会在Mysql重新启动时丢失。



