---
title: User yangql is not allowed to impersonate anonymous
date: 2017-02-26 15:54:59
tags: beeline
categories: Hive
---
使用HiveServer2 and Beeline模式运行时，启动好HiveServer后运行
```
beeline -u jdbc:hive2://localhost:10000 -n root
```
连接server时出现错误:
```
java.lang.RuntimeException: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.security.authorize.AuthorizationException):
User yangql is not allowed to impersonate anonymous
```
<!-- more -->
修改hadoop 配置文件 etc/hadoop/core-site.xml,加入如下配置项【其中yangql是用户名，可以是自己的用户名如zhangsan】:
```
<property>
    <name>hadoop.proxyuser.yangql.hosts</name>
    <value>*</value>
</property>
<property>
    <name>hadoop.proxyuser.yangql.groups</name>
    <value>*</value>
</property>
```  
重启hadoop
