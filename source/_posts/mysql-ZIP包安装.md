---
title: mysql-ZIP包安装
date: 2017-03/11 14:52:31
tags: mysql
categories: mysql
---
1.先将dos目录改成mysql的bin目录
```
cd D:\Program Files\mysql-5.7.17\bin>
```
2.修改mysql目录下的my-default.ini文件
找到下面两行改成你的mysql目录
```
basedir=D:\Program Files\mysql-5.7.17\ （mysql所在目录）
datadir=D:\Program Files\mysql-5.7.17\data （mysql所在目录\data）
```
3.命令行：mysqld install
```
D:\Program Files\mysql-5.7.17\bin>mysqld.exe install
```
4.直接启动看看----net start mysql（失败了）
```
D:\Program Files\mysql-5.7.17\net start mysql
MySQL 服务正在启动 .
MySQL 服务无法启动。
```

5.执行一下命令
```
D:\Program Files\mysql-5.7.17\bin>mysqld --initialize-insecure
```
6.设置密码
```
D:\Program Files\mysql-5.7.17\bin>mysql -uroot
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.17 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set password="root"
    -> ;
Query OK, 0 rows affected (0.00 sec)

mysql>
```
