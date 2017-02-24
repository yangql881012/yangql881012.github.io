---
title: Linux下MySQL安装
date: 2017-01-19 12:04:31
tags: Linux,MySQL
categories: MySQL
---
## 1.MySQL相关介绍 ##
MySQL是一个关系型数据库管理系统，由瑞典MySQL AB公司开发，目前属于Oracle公司。MySQL是一种关联数据库管理系统，关联数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。MySQL的SQL语言是用于访问数据库的最常用标准化语言。MySQL软件采用了双授权政策，它分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择MySQL作为网站数据库。在Linux上安装mysql数据库，我们可以去其官网上下载mysql数据库的rpm包，http://dev.mysql.com/downloads/mysql/5.6.html#downloads。
<!-- more -->
本教程通过yum来进行MySQL数据库的安装的，通过这种方式进行安装，可以将跟MySQL相关的一些服务、jar包都给我们安装好，省去了很多不必要的麻烦。
## 2.卸载掉原有mysql ##
- 查看该操作系统上是否已经安装了mysql数据库
```
[root@hadoop01 ~]# rpm -qa | grep mysql
mysql-server-5.1.73-7.el6.x86_64
mysql-libs-5.1.73-7.el6.x86_64
mysql-5.1.73-7.el6.x86_64
mysql-devel-5.1.73-7.el6.x86_64
```
- 通过 rpm -e 命令 或者 rpm -e --nodeps 命令来卸载掉
```
[root@hadoop01 ~]# rpm -e mysql　　// 普通删除模式
[root@hadoop01 ~]# rpm -e --nodeps mysql　　// 强力删除模式，如果使用上面命令删除时，提示有依赖的其它文件，则用该命令可以对其进行强力删除
```  

## 3.通过yum来进行MySQL的安装 ##  
- 输入 yum list | grep mysql 命令来查看yum上提供的mysql数据库可下载的版本：
```
[root@hadoop01 ~]# yum list | grep mysql
mysql.x86_64                                5.1.73-7.el6                @base   
mysql-devel.x86_64                          5.1.73-7.el6                @base   
mysql-libs.x86_64                           5.1.73-7.el6                @base   
mysql-server.x86_64                         5.1.73-7.el6                @base   
apr-util-mysql.x86_64                       1.3.9-3.el6_0.1             base    
bacula-director-mysql.x86_64                5.0.0-13.el6                base    
bacula-storage-mysql.x86_64                 5.0.0-13.el6                base    
dovecot-mysql.x86_64                        1:2.0.9-22.el6              base    
freeradius-mysql.x86_64                     2.2.6-6.el6_7               base    
libdbi-dbd-mysql.x86_64                     0.8.3-5.1.el6               base    
mod_auth_mysql.x86_64                       1:3.0.0-11.el6_0.1          base    
mysql-bench.x86_64                          5.1.73-7.el6                base    
mysql-connector-java.noarch                 1:5.1.17-6.el6              base    
mysql-connector-odbc.x86_64                 5.1.5r1144-7.el6            base    
mysql-devel.i686                            5.1.73-7.el6                base    
mysql-embedded.i686                         5.1.73-7.el6                base    
mysql-embedded.x86_64                       5.1.73-7.el6                base    
mysql-embedded-devel.i686                   5.1.73-7.el6                base    
mysql-embedded-devel.x86_64                 5.1.73-7.el6                base    
mysql-libs.i686                             5.1.73-7.el6                base    
mysql-test.x86_64                           5.1.73-7.el6                base    
pcp-pmda-mysql.x86_64                       3.10.9-6.el6                base    
php-mysql.x86_64                            5.3.3-48.el6_8              updates
qt-mysql.i686                               1:4.6.2-28.el6_5            base    
qt-mysql.x86_64                             1:4.6.2-28.el6_5            base    
rsyslog-mysql.x86_64                        5.8.10-10.el6_6             base    
rsyslog7-mysql.x86_64                       7.4.10-5.el6                base    
```
- 输入 yum install -y mysql-server mysql mysql-devel 命令将mysql mysql-server mysql-devel都安装好
```
[root@hadoop01 ~]# yum install -y mysql-server mysql mysql-deve
```
- 查看刚安装好的mysql-server的版本
```
[root@hadoop01 ~]# rpm -qi mysql-server
```

## 4.MySQL数据库的初始化及相关配置 ##
我们在安装完mysql数据库以后，会发现会多出一个mysqld的服务，这个就是咱们的数据库服务，我们通过输入 `service mysqld start` 命令就可以启动我们的mysql服务。注意：如果我们是第一次启动mysql服务，mysql服务器首先会进行初始化的配置，如：
```
[root@hadoop01 ~]# service mysqld start

初始化 MySQL 数据库： WARNING: The host 'xiaoluo' could not be looked up with resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MySQL version. The MySQL daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MySQL privileges !
Installing MySQL system tables...
OK
Filling help tables...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/bin/mysqladmin -u root password 'new-password'
/usr/bin/mysqladmin -u root -h xiaoluo password 'new-password'

Alternatively you can run:
/usr/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:
cd /usr ; /usr/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl
cd /usr/mysql-test ; perl mysql-test-run.pl

Please report any problems with the /usr/bin/mysqlbug script!

正在启动 mysqld：                                          ```

这时我们会看到第一次启动MySQL服务器以后会提示非常多的信息，目的就是对MySQL数据库进行初始化操作，当我们再次重新启动MySQL服务时，就不会提示这么多信息了
```
[root@hadoop01 ~]# service mysqld restart
Stopping mysqld:                                           [  OK  ]
Starting mysqld:                                           [  OK  ]
[root@hadoop01 ~]#
```

我们在使用MySQL数据库时，都得首先启动mysqld服务，我们可以 通过  `chkconfig --list | grep mysqld` 命令来查看mysql服务是不是开机自动启动，如：
```
[root@hadoop01 ~]# chkconfig --list | grep mysqld
mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@hadoop01 ~]#
````

我们发现mysqld服务并没有开机自动启动，我们当然可以通过 `chkconfig mysqld on` 命令来将其设置成开机启动，这样就不用每次都去手动启动了
```
[root@hadoop01 ~]# chkconfig mysqld on
[root@hadoop01 ~]# chkconfig --list | grep mysql
mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
[root@hadoop01 ~]#
```

为root用户设置密码
```
[root@hadoop01 ~]# mysqladmin -u root password 'root'　　// 通过该命令给root账号设置密码为 root

```
登录
```
[root@hadoop01 ~]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
## 5.MySQL数据库的主要配置文件 ##  

- /etc/my.cnf 这是mysql的主配置文件  

```
[root@hadoop01 ~]# cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
[root@hadoop01 ~]#
```
- /var/lib/mysql   mysql数据库的数据库文件存放位置
```
[root@hadoop01 ~]# ls -lrt /var/lib/mysql
total 20488
drwx------ 2 mysql mysql     4096 Jan 19 04:59 test
drwx------ 2 mysql mysql     4096 Jan 19 04:59 mysql
-rw-rw---- 1 mysql mysql  5242880 Jan 19 04:59 ib_logfile1
-rw-rw---- 1 mysql mysql 10485760 Jan 19 07:21 ibdata1
-rw-rw---- 1 mysql mysql  5242880 Jan 19 07:21 ib_logfile0
srwxrwxrwx 1 mysql mysql        0 Jan 19 07:21 mysql.sock
[root@hadoop01 ~]#
```
- 创建数据库
```
mysql> create database yangql
    -> ;
Query OK, 1 row affected (0.00 sec)

```  
-  /var/log MySQL数据库的日志输出存放位置  


因为我们的mysql数据库是可以通过网络访问的，并不是一个单机版数据库，其中使用的协议是 tcp/ip 协议，我们都知道mysql数据库绑定的端口号是 3306 ，所以我们可以通过 netstat -anp 命令来查看一下，Linux系统是否在监听 3306 这个端口号：
```

```
