---
title: sqoop实战
date: 2017-01-23 00:54:59
tags: sqoop
categories: sqoop
---
sqoop连接mysql数据库，讲数据库表中数据导出到HDFS。
<!-- more -->
 ## 1. MySQL数据库准备 ##
 （1）创建数据库
 ```
 [yangql@hadoop01 ~]$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE yangql DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
ERROR 1007 (HY000): Can't create database 'yangql'; database exists
mysql> CREATE DATABASE yangql01 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
Query OK, 1 row affected (0.02 sec)

mysql>
 ```
 （2）创建表  
 ```
 CREATE TABLE `persinfo` (
  `id` int(4) NOT NULL AUTO_INCREMENT,
  `name` char(20) NOT NULL,
  `sex` int(4) NOT NULL DEFAULT '0',
  `last_ts` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
);
 ```
 （3）插入数据  
 ```
INSERT INTO `persinfo` VALUES (1, 'fsafsaf', 1, '2017-1-21 02:46:24');
INSERT INTO `persinfo` VALUES (2, '111', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (3, 'Tom', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (4, 'Tom', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (5, 'Tom', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (6, 'Tom', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (7, 'Tom', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (8, 'Tom', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (9, 'mark', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (10, 'safada', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (11, 'fsafdsa', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (12, '44', 0, '2017-1-19 10:27:15');
INSERT INTO `persinfo` VALUES (13, 'fsfffff', 0, '2017-1-20 10:35:44');
INSERT INTO `persinfo` VALUES (14, '4444', 1, '2017-1-22 02:46:08');
 ```
 ## 2 导入  ##
 （1）通用参数  
--connect，数据库连接字符串  
--username，数据库访问用户名  
--password ，指定数据库连接密码（明文）  
--P 交互式的指定数据库密码  
--password-file，使用密码文件制定数据库密码  
（2）导入控制参数——选择部分数据导入
--query，要导入的数据用SQL查询控制  
示例：  
```sqoop import --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --query "select * from persinfo where id<=5 AND $CONDITION"```  
--table:指定要导入的数据表     --where:指定导入时的条件 "id<=5"    
–-columns:指定要导入的字段名 例如id,name  
（3）目的目录（HDFS）
--warehouse-dir:指定导入到HDFS中的目录  
（4）分隔符
--fields-terminated-by:例– fields-terminated-by '|'，默认情况下用','进行字段分隔
--lines-terminated-by,行分隔符
--escaped-by，转义字符  
（5）控制导入并行度
--m或--num-mappers [INT]:控制导入时MR作业的Map任务数量，后接一个整数值，用来表示MR的并行度。在进行并行导入的时候，Sqoop会使用split-by进行负载切分（按照表的PK进行切分），首先获取切分字段Max和Min值，再根据Max和Min值去进行切分，举例：id [1,1000]，Sqoop会直接使用4个Map任务"select * from persinfo where id >=min and id <=max，(1,250),(250,500),(500,750),(750,1000)。如果非常不幸，你的ID或切分字段不是均匀分布的话，会导致任务的不平衡。注明：Sqoop目前不能使用多个字段作为切分字段。  
（6）类型映射(导入到Hive时使用)
--map-column-hive， id=String,value=Int  
## 3 导入实例 ##
（1）查看对应库、表情况  
```
sqoop list-databases --connect jdbc:mysql://hadoop01:3306 --username root --password root  
```  
```
sqoop list-tables --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root
```  
（2）导入表  
```
sqoop import --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --table persinfo
```
注:
- <font color="red">目录不能已存在</font>
- <font color="red">没指定-m或splite-by时，表必须有PK</font>  

（3）指定导入目录  
```
sqoop import --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --table persinfo --warehouse-dir /input
```
（4）控制并行度
```
sqoop import --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --table persinfo -m 2
```
（5）控制字段分隔符
```
sqoop import --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --table persinfo -m 2 –fields-terminated-by "|"
```
（6）导入部分数据  
仅导入id<=5的数据
--query，必须使用--target-dir指定导入目录，且删除--table参数
```
sqoop import --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --query "select * from persinfo where id<=5 and \$CONDITIONS" -m 1 --target-dir /user/yangql/persinfo
```
--where，直接用--where指定条件，用引号引起来
```
sqoop import --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --table persinfo --where "id<=5" -m 1
```
--columns，指定导入字段名
仅导入id,name
```
sqoop import --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --table persinfo --columns id,name -m 1
```
（7）使用文件进行导入

注意：文件中的参数与指定值必须每个一行
写入参数文件sqoop.im，内容如下：
```
import  
--connect  
jdbc:mysql://hadoop01:3306/yangql  
--username  
root  
--password  
root  
--table  
persinfo  
--columns  
id,name
-m
1
```
执行sqoop：
```
sqoop --options-file sqoop.im
```
## 4 导出控制参数 ##  
--columns: id,name
注意：没有被包含在--columns后面（例如sex）的这些列名或字段要么具备默认值，要么就允许插入空值，数据库会拒绝接受sqoop导出的数据，导致Sqoop作业失败  
--export-dir:导出目录，在执行导出的时候，必须指定这个参数，同时需要具备--table或--call参数两者之一，--table是指的导出数据库当中对应的表，--call是指的某个存储过程  
--input-null-string、--input-null-non-string:这两参数是可选，如果没有指定第一个参数，对于字符串类型的列来说，“NULL”这个字符串就回被翻译成空值，如果没有使用第二个参数，无论是“NULL”字符串还是说空字符串也好，对于非字符串类型的字段来说，这两个类型的空串都会被翻译成空值      导出实例:  
```
sqoop export --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --table persinfo --export-dir /user/yangql/persinfo/
```  
## 5 更新导出 ##
- --update-key:更新标识，即根据某个字段进行更新，例如id，可以指定多个更新标识的，用逗号分隔   
- --updatemod:有两种模式，一种是updateonly（默认模式），仅仅更新已存在的数据记录，不会插入新纪录，另一种模式是allowinsert，允许插入新纪录  
- Sqoop的Export工具，对应两种语句，一种是Insert语句，如果表当中存在PK约束，且表中已包含数据，此时，导出报错。此时需要用到—-update-key和--updatemod  
- 如果指定了update-key，那么Sqoop就会修改在数据表中已存在的数据，此时的每一个更行数据记录都会变成一个Update语句，用这个语句去更新目标表中已存在的数据，这是根据--update-key所指定的这个列进行更新的。Update set name='tom' where id = 1。
- 若update-key所指定的字段不是PK字段，若同时updatemod使用updateonly模式时，就仅进行更新，若updatemod使用allowinsert模式，那么实质上就是一个insert操作
- 若update-key所指定的字段是PK字段，同时updatemod是allowinsert时，实质上是一个insert & update的操作，若updatemod是updateonly时，实质仅为update操作  

```
# updateonly example
sqoop export --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --table persinfo --export-dir /user/yangql/persinfo/ --update-key class_id --update-mode updateonly

# allowinsert example
sqoop export --connect jdbc:mysql://hadoop01:3306/yangql --username root --password root --table persinfo --export-dir /user/yangql/persinfo/ --update-key class_id --update-mode allowinsert
```  
## 6 增量导入 ##
（1）核心参数  
--check-column:用来指定一些列，这些列在导入时用来检查做决定数据是否要被作为增量数据，在一般关系型数据库中，都存在类似Last_Mod_TM的字段或主键。注意：这些被检查的列的类型不能是任意字符类型，例如Char，VARCHAR…（即字符类型不能作为增量标识字段）   
--incremental:用来指定增量导入的模式（Mode），append和lastmodified
--last-value:指定上一次导入中检查列指定字段最大值  

(2)增量模式（Mode）
append:在导入的新数据ID值是连续时采用，对数据进行附加   
lastmodified:在源表中有数据更新的时候使用，检查列就必须是一个时间戳或日期类型的字段，更新完之后，last-value会被设置为执行增量导入时的当前系统时间
- Append:加不加--last-value的区别在于：数据是否冗余，如果不加，则会导入源表中的所有数据导致数据冗余。
- Lastmodified:当使用--incremental lastmodified模式进行导入且导入目录已存在时，需要使用–-merge-key或–-append
导入>=last-value的值。

（3）示例
```
sqoop import --connect jdbc:mysql://hadoop01:3306/yangql --table persinfo --username root --password root --check-column last_mod_ts --incremental lastmodified --last-value "2017-01-03 22:39:43" --merge-key id -m 1
```  
## 7 sqoop作业 ##
（1）创建作业
```
sqoop job --create job1 \
--import \
--connect jdbc:mysql://hadoop01:3306/yangql \
--username root \
--table root --m 1
```  
（2）验证作业

```
yangql@hadoop01 ~]$ sqoop job --list
Warning: /home/yangql/app/sqoop-1.4.6/../hbase does not exist! HBase imports will fail.
Please set $HBASE_HOME to the root of your HBase installation.
Warning: /home/yangql/app/sqoop-1.4.6/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/yangql/app/sqoop-1.4.6/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
Warning: /home/yangql/app/sqoop-1.4.6/../zookeeper does not exist! Accumulo imports will fail.
Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation.
17/01/23 02:25:54 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
Available jobs:
  job1
[yangql@hadoop01 ~]$
```
（3）检查作业

```
[yangql@hadoop01 ~]$ sqoop job --show job1
Warning: /home/yangql/app/sqoop-1.4.6/../hbase does not exist! HBase imports will fail.
Please set $HBASE_HOME to the root of your HBase installation.
Warning: /home/yangql/app/sqoop-1.4.6/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /home/yangql/app/sqoop-1.4.6/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
Warning: /home/yangql/app/sqoop-1.4.6/../zookeeper does not exist! Accumulo imports will fail.
Please set $ZOOKEEPER_HOME to the root of your Zookeeper installation.
17/01/23 02:26:48 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
Enter password:
Job: job1
Tool: import
Options:
----------------------------
verbose = false
incremental.last.value = 2017-01-21 04:13:42.0
db.connect.string = jdbc:mysql://hadoop01:3306/yangql
codegen.output.delimiters.escape = 0
codegen.output.delimiters.enclose.required = false
codegen.input.delimiters.field = 0
hbase.create.table = false
db.require.password = true
hdfs.append.dir = true
db.table = persinfo
codegen.input.delimiters.escape = 0
import.fetch.size = null
accumulo.create.table = false
codegen.input.delimiters.enclose.required = false
db.username = root
reset.onemapper = false
codegen.output.delimiters.record = 10
import.max.inline.lob.size = 16777216
hbase.bulk.load.enabled = false
hcatalog.create.table = false
db.clear.staging.table = false
incremental.col = last_ts
codegen.input.delimiters.record = 0
enable.compression = false
hive.overwrite.table = false
hive.import = false
codegen.input.delimiters.enclose = 0
accumulo.batch.size = 10240000
hive.drop.delims = false
codegen.output.delimiters.enclose = 0
hdfs.delete-target.dir = false
codegen.output.dir = .
codegen.auto.compile.dir = true
relaxed.isolation = false
mapreduce.num.mappers = 1
accumulo.max.latency = 5000
import.direct.split.size = 0
codegen.output.delimiters.field = 44
export.new.update = UpdateOnly
incremental.mode = DateLastModified
hdfs.file.format = TextFile
codegen.compile.dir = /tmp/sqoop-yangql/compile/15610058cdad963093623efd00897f90
direct.import = true
hdfs.target.dir = /input/yangql
hive.fail.table.exists = false
db.batch = false
[yangql@hadoop01 ~]$
```
（4）执行作业
```
sqoop job --exec job1
```
