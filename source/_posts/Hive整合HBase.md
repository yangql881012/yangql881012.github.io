---
title: Hive整合HBase
date: 2017/06/06 14:52:31
tags: Hive
toc: true
categories: 大数据技术
---

Hive和Hbase有各自不同的特征：hive是高延迟、结构化和面向分析的，hbase是低延迟、非结构化和面向编程的。Hive数据仓库在hadoop上是高延迟的。Hive集成Hbase就是为了使用hbase的一些特性。  
Hive继承HBase可以有效利用HBase数据库的存储特性，如行更新和列索引等。在集成的过程中注意维持HBase jar包的一致性。Hive集成HBase需要在Hive表和HBase表之间建立映射关系，也就是Hive表的列和列类型与HBase表的列族及列限定词建立关联。每一个在Hive表中的域都存在与HBase中，而在Hive表中不需要包含所有HBase中的列。HBase中的rowkey对应到Hive中为选择一个域使用 :key 来对应，列族(cf映射到Hive中的其他所有域，列为(cf:cq)。
<!-- more -->

## 1.创建Hive中与HBase中对应的表 ##
```
CREATE  TABLE crm.user1 (
rowkey string,
info map<STRING,STRING>
) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,info:")
TBLPROPERTIES ("hbase.table.name" = "crm:user1");
```
## 2.创建Hbase表 ##
crm:命名空间
```
create 'crm:user1',{NAME => 'info',VERSIONS => 1}

向user表中插入一些数据：
put 'crm:user1','1','info:name','zhangsan'
put 'crm:user1','1','info:age','25'
put 'crm:user1','2','info:name','lisi'
put 'crm:user1','2','info:age','22'
put 'crm:user1','3','info:name','wangswu'
put 'crm:user1','3','info:age','21'
```
HBase查看数据  
```
scan 'crm:user1'
hbase(main):002:0> scan 'crm:user1'
ROW                                                                  COLUMN+CELL                                                                                                                                                                                              
 1                                                                   column=info:age, timestamp=1496762712287, value=25                                                                                                                                                       
 1                                                                   column=info:name, timestamp=1496762712252, value=zhangsan                                                                                                                                                
 2                                                                   column=info:age, timestamp=1496762712375, value=22                                                                                                                                                       
 2                                                                   column=info:name, timestamp=1496762712312, value=lisi                                                                                                                                                    
 3                                                                   column=info:age, timestamp=1496762713400, value=21                                                                                                                                                       
 3                                                                   column=info:name, timestamp=1496762712414, value=wangswu                                                                                                                                                 
3 row(s) in 0.0920 seconds
```
Hive查看数据
```
select * from crm.user1;
hive (default)> select * from crm.user1;
OK
user1.rowkey	user1.info
1	{"age":"25","name":"zhangsan"}
2	{"age":"22","name":"lisi"}
3	{"age":"21","name":"wangswu"}
Time taken: 0.139 seconds, Fetched: 3 row(s)
```
## 3.hive插入数据到hbase ##
```
INSERT INTO TABLE crm.user1
SELECT '4' AS rowkey,
map('name','lijin','age','22') AS info
from dual limit 1;
```

## 4.从Hive中创建HBase表 ##
使用HQL语句创建一个指向HBase的Hive表  
```
CREATE TABLE hbase_table_1(key int, value string) //Hive中的表名hbase_table_1
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'  //指定存储处理器
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf1:val") //声明列族，列名
TBLPROPERTIES ("hbase.table.name" = "xyz", "hbase.mapred.output.outputtable" = "xyz");  
//hbase.table.name声明HBase表名，为可选属性默认与Hive的表名相同，
//hbase.mapred.output.outputtable指定插入数据时写入的表，如果以后需要往该表插入数据就需要指定该值

CREATE TABLE hbase_table_1(key int, value string)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'  
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf1:val")
TBLPROPERTIES ("hbase.table.name" = "hbase_table_1", "hbase.mapred.output.outputtable" = "hbase_table_1");  
```
通过HBase shell可以查看刚刚创建的HBase表的属性  
```
hbase(main):002:0> desc 'hbase_table_1'
Table hbase_table_1 is ENABLED                                                                                                                                              
hbase_table_1                                                                                                                                                               
COLUMN FAMILIES DESCRIPTION                                                                                                                                                 
{NAME => 'cf1', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', COMPRESSION => 'NONE', TTL => 'FO
REVER', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                                                                          
1 row(s) in 0.4100 seconds

hbase(main):003:0>
```
## 5.从Hive中映射HBase ##
创建一个指向已经存在的HBase表的Hive表  
```
CREATE EXTERNAL TABLE hbase_table_2(key int, value string)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = "cf1:val")
TBLPROPERTIES("hbase.table.name" = "some_existing_table", "hbase.mapred.output.outputtable" = "some_existing_table");
```
- 该Hive表一个外部表，所以删除该表并不会删除HBase表中的数据
注意
- 建表或映射表的时候如果没有指定:key则第一个列默认就是行键
- HBase对应的Hive表中没有时间戳概念，默认返回的就是最新版本的值
- 由于HBase中没有数据类型信息，所以在存储数据的时候都转化为String类型

## 6.多列及多列族的映射 ##
如下表：value1和value2来自列族a对应的b c列，value3来自列族d对应的列
```
CREATE TABLE hbase_table_1(key int, value1 string, value2 int, value3 int)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES (
"hbase.columns.mapping" = ":key,a:b,a:c,d:e"
);
```
## 7.Hive Map类型在HBase中的映射规则 ##
通过Hive的Map数据类型映射HBase表，这样每行都可以有不同的列组合，列名与map中的key对应，列值与map中的value对应
```
CREATE TABLE hbase_table_1(value map<string,int>, row_key int)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES (
"hbase.columns.mapping" = "cf:,:key"
);
```
