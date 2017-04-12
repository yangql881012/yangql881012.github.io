---
title: hadoop2-hdfs-垃圾箱简介
date: 2016-11-13 09:09:59
tags: Hadoop2，垃圾箱
toc: true
categories: Hadoop
---
hdfs为每一个用户创建一个回收站：
目录： /user/用户名/.Trash/ 每一个被用户通过shell删除的文件/目录
在系统回收站中都有一个周期，周期过后hdfs会自动将这些数据彻底删除
周期内 可以被用户恢复。
<!-- more -->
- 回收站目录如下：
```
[yangql@hadoop01 ~]$ hadoop fs -ls /user/yangql/.Trash/
Found 1 items
drwx------   - yangql supergroup          0 2017-01-18 03:58 /user/yangql/.Trash/Current
[yangql@hadoop01 ~]$ hadoop fs -ls /user/yangql/.Trash/Current
Found 1 items
-rw-r--r--   2 yangql supergroup          5 2017-01-18 03:58 /user/yangql/.Trash/Current/my.txt
[yangql@hadoop01 ~]$
```
- 配置垃圾箱(修改core-site.xml文件)
```
<property>
<name>fs.trash.interval</name>
<value>1440</value>
</property>
```
- 将回收站数据返回hdfs中：
```
[yangql@hadoop01 ~]$ hdfs dfs -mv /user/yangql/.Trash/Current/my.txt /
```
- Java端删除文件
```
//删除  将文件放在回收站中
  final Trash trash = new Trash(fileSystem, fileSystem.getConf()); //创建回收站
  trash.moveToTrash(new Path("/dir1/file1"));// 将文件放在回收站中
  fileSystem.delete(new Path("/dir1/file1"), true);// 删除文件 hdfs删除文件是直接删掉 不会进入回收站 上面操作是将文件先拷贝到回收站在去删除该文件
```
