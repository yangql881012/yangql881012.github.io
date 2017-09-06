---
title: Linux下mysql-5.7.9-linux-glibc2.5-x86_64.tar.gz安装
date: 2017-01-19 12:04:31
tags: MySQL
toc: true
categories: 大数据技术
---
1.从官网下载 mysql-5.7.9-linux-glibc2.5-x86_64.tar.gz
```
官网： http://dev.mysql.com/downloads/mysql/
wget -c http://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.9-linux-glibc2.5-x86_64.tar.gz
```
2.创建mysql的用户组/用户, data目录及其用户目录
```
# groupadd mysql
# useradd -g mysql -d /home/mysql mysql
# mkdir /home/mysql/data
```
<!-- more -->
3.解压安装包并将解压包里的内容拷贝到mysql的安装目录/home/mysql
```
# tar -xzvf mysql-5.7.9-linux-glibc2.5-x86_64.tar.gz
# cd mysql-5.7.9-linux-glibc2.5-x86_64
# mv * /home/mysql
```
4.初始化mysql数据库
```
# cd /home/mysql/data
# rm -fr *
# ./bin/mysqld --user=mysql --basedir=/home/mysql --datadir=/home/mysql/data --initialize
2016-04-08T01:47:57.556677Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2016-04-08T01:47:59.945537Z 0 [Warning] InnoDB: New log files created, LSN=45790
2016-04-08T01:48:00.333528Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2016-04-08T01:48:00.434908Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: ece26421-fd2b-11e5-a1e3-00163e001e5c.
2016-04-08T01:48:00.440125Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2016-04-08T01:48:00.440904Z 1 [Note] A temporary password is generated for root@localhost: **mjT,#x_5sW
```
牢记上面的随机密码， 如上dkdsfd*dmode, 下面我们修改密码时需要用到。

5.检测下是否能启动mysql服务
```
# cd /home/mysql
# ./support-files/mysql.server start
Starting MySQL.. SUCCESS!
```
若改用了/home/mysql为mysql的安装目录basedir， 则在启动服务时会出现如下错误：
```
# ./support-files/mysql.server start
./support-files/mysql.server: line 276: cd: /usr/local/mysql: No such file or directory
Starting MySQL ERROR! Couldn't find MySQL server (/usr/local/mysql/bin/mysqld_safe)
```
由上面可知mysql的tar.gz安装包的默认安装目录为/usr/local/mysql， 这时候我们需要修改/support-files/mysql.server文件的basedir和datadir目录路径为我们环境所在的mysql的basedir和datadir路径， 如下：
```
# vim support-files/mysql.server
--------------------------
...
basedir=/home/mysql
datadir=/home/mysql/data
...
--------------------------
# ./support-files/mysql.server start
Starting MySQL.. SUCCESS!
```
6.创建软链接
```
# ln -s /home/mysql/bin/mysql /usr/bin/mysql
```
7.创建配置文件

将默认生成的my.cnf备份
```
# mv /etc/my.cnf /etc/my.cnf.bak
```
进入mysql的安装目录支持文件目录
```
# cd /home/mysql/support-files
```
拷贝配置文件模板为新的mysql配置文件,
```
# cp my-default.cnf /etc/my.cnf
```
可按需修改新的配置文件选项， 不修改配置选项， mysql则按默认配置参数运行.
如下是我修改配置文件/etc/my.cnf， 设置编码为utf8以防乱码
```
# vim /etc/my.cnf

[mysqld]

basedir = /home/mysql
datadir = /home/mysql/data

character_set_server=utf8
init_connect='SET NAMES utf8'


[client]
default-character-set=utf8
```
8.配置mysql服务开机自动启动

拷贝启动文件到/etc/init.d/下并重命令为mysqld
```
# cp /home/mysql/support-files/mysql.server /etc/init.d/mysqld
```
增加执行权限
```
# chmod 755 /etc/init.d/mysqld
```
检查自启动项列表中没有mysqld这个，如果没有就添加mysqld：
```
# chkconfig --list mysqld
# chkconfig --add mysqld
```
设置MySQL在345等级自动启动
```
# chkconfig --level 345 mysqld on
```
或用这个命令设置开机启动：
```
# chkconfig mysqld on
```
9.mysql服务的启动/重启/停止

启动mysql服务
```
# service mysqld start
```
重启mysql服务
```
# service mysqld restart
```
停止mysql服务
```
# service mysqld stop
```
10.初始化mysql用户root的密码

先将mysql服务停止
```
# service mysqld stop
```
进入mysql安装目录， 执行：
```
# cd /home/mysql
# ./bin/mysqld_safe --skip-grant-tables --skip-networking&
[1] 6225
[root@localhost mysql]# 151110 02:46:08 mysqld_safe Logging to '/home/mysql/data/localhost.localdomain.err'.
151110 02:46:08 mysqld_safe Starting mysqld daemon with databases from /home/mysql/data
```
另外打开一个终端(p.s. 如果是ssh连接登录的, 另外创建一个ssh连接即可）， 执行操作如下：
```
# mysql -u root mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.9 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use mysql;
Database changed
mysql> UPDATE user SET password=PASSWORD('123456') WHERE user='root';
ERROR 1054 (42S22): Unknown column 'password' in 'field list'
mysql> update user set authentication_string = PASSWORD('123456') where user = 'root';
Query OK, 1 row affected, 1 warning (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> \s
--------------
mysql  Ver 14.14 Distrib 5.7.9, for linux-glibc2.5 (x86_64) using  EditLine wrapper

Connection id:      2
Current database:   mysql
Current user:       root@
SSL:            Not in use
Current pager:      stdout
Using outfile:      ''
Using delimiter:    ;
Server version:     5.7.9 MySQL Community Server (GPL)
Protocol version:   10
Connection:     Localhost via UNIX socket
Server characterset:    utf8
Db     characterset:    utf8
Client characterset:    utf8
Conn.  characterset:    utf8
UNIX socket:        /tmp/mysql.sock
Uptime:         4 min 47 sec

Threads: 1  Questions: 43  Slow queries: 0  Opens: 127  Flush tables: 1  Open tables: 122  Queries per second avg: 0.149
--------------

mysql> exit;
Bye
```
到此， 设置完mysql用户root的密码且确保mysql编码集是utf8, 注意上面， 新版本的mysql.user表里的密码字段是authentication_string

快捷键ctrl + c停止# ./bin/mysqld_safe ...命令， 重新启动mysql服务， 用新密码连接mysql：
```
# service mysqld start
Starting MySQL SUCCESS!
[root@localhost bin]# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.9

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use mysql;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql > exit;
Bye

咦？又要我改密码， 我们通过mysqladmin来修改密码， 先输入原密码， 再设置新密码， 总算可以了吧！！！

# cd /home/mysql
# ./bin/mysqladmin -u root -p password
Enter password:
New password:
Confirm new password:
Warning: Since password will be sent to server in plain text, use ssl connection to ensure password safety.
# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.9 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>

或直接：

# ./bin/mysqladmin -uroot -p'**mjT,#x_5sW' password '123456'
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
Warning: Since password will be sent to server in plain text, use ssl connection to ensure password safety.
其中， **mjT,#x_5sW就是我们在使用mysqld --initialize时牢记下的随机密码
```
11.mysql远程授权

格式如下：
``
mysql> grant all [privileges] on db_name.table_name to 'username'@'host' identified by 'password';
示例如下：

mysql> grant all privileges on *.* to 'root'@'%' identified by '123456';
Query OK, 0 rows affected, 1 warning (0.04 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql>
或用

mysql> grant all on *.* to 'root'@'%' identified by '123456';
```
到此， 完成了mysql的安装 及配置！！！
