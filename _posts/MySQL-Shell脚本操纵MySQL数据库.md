## -e选项

```bash
mysql -uroot -p123456 -e 'show databases;select user,host from mysql.user'
```
## <<EOF

```bash
mysql -uroot -p123456 <<EOF

> show databases;
> select user,host from mysql.user;
> EOF
```
例题
> MySQL中有一个test数据库，里面有数十张‘tbllog_’为前序的表，现要求除了tbllog_pay、tbllog_role、tbllog_online表外，其他的全部进行清空，请写一个shell脚本？

```bash
#!/bin/bash

for i in $(mysql -uroot -p123456 -e 'show tables from test;' 2>/dev/null|sed 1d|egrep -v 'tbllog_online|tbllog_pay|tbllog_role')
#将密码提示警告信息重定向到黑洞文件
do
        mysql -uroot -p123456 -e "truncate test.$i" 2>/dev/null
done
        echo '清空完成'
```
