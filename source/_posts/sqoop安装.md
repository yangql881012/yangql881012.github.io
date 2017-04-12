---
title: sqoop安装
date: 2017-01-18 00:54:59
tags: sqoop
toc: true
categories: sqoop
---
Sqoop是SQL to Hadoop的缩写，主要作用在于在结构的数据存储(关系型数据库)与Hadoop之间进行数据双向交换。也就是说，Sqoop可以将关系数据库(如MySQL,Oracel等)的数据导入Hadoop的HDFS、Hive中，也可以将HDFS、Hive的数据导出到关系数据库中。Sqoop充分利用了Hadoop的优点，整个导入都是由MapReduce计算框架实现并行化，非常高效。
<!-- more -->
(1) 解压文件  
[yangql@hadoop01 opt]$ tar -xvf sqoop-1.4.6.binhadoop-2.0.4-alpha.tar.gz  
(2) 将文件重命名  
[yangql@hadoop01 opt]$ mv sqoop-1.4.6.binhadoop-2.0.4-alpha sqoop-1.4.6  
(3) 配置环境变量  
export SQOOP_HOME=/home/yangql/app/sqoop-1.4.6
export PATH=PATH:PATH:SQOOP_HOME/bin  
(4) 验证(忽略警告信息)
```
[yangql@hadoop01 bin]$ sqoop version
Warning: /home/yangql/app/sqoop-1.4.6/../hbase does not exist! HBase imports will fail.
Please set $HBASE_HOME to the root of your HBase installation.
Warning: /home/yangql/app/sqoop-1.4.6/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/yangql/app/sqoop-1.4.6/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
Warning: /home/yangql/app/sqoop-1.4.6/../zookeeper does not exist! Accumulo imports will fail.
Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation.
Try 'sqoop help' for usage.
```
