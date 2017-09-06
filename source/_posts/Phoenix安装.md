---
title: Phoenix安装
date: 2017-06-26 12:54:59
tags: Phoenix
toc: true
categories: 大数据技术
---
## 1.解压并配置 ##
将下载好的安装包上传到我们的主节点上
```
tar -xvf apache-phoenix-4.10.0-HBase-1.2-bin.tar.gz
mv apache-phoenix-4.10.0-HBase-1.2-bin/ phoenix-4.10
```
```
#config phoenix4.10
export PHOENIX_HOME=/home/yangql/app/phoenix-4.10
export PHOENIX_CLASSPATH=$PHOENIX_HOME
export PATH=$PATH:$PHOENIX_HOME/bin
```
<!-- more -->
## 2.配置hbase ##
进入到phoenix的安装目录，找到 “phoenix-4.10.0-HBase-1.2-server.jar” ，将这个 jar 包拷贝到集群中每个节点( 主节点也要拷贝 )的 hbase 的 lib 目录下

## 3.重启hbase ##
```
sh /home/yangql/app/hbase-1.2.5/bin/stop-hbase.sh
```

## 4.启动phoenix ##
```
[yangql@hadoop01 bin]$ python sqlline.py hadoop01,hadoop02,hadoop03:2181
Setting property: [incremental, false]
Setting property: [isolation, TRANSACTION_READ_COMMITTED]
issuing: !connect jdbc:phoenix:hadoop01,hadoop02,hadoop03:2181 none none org.apache.phoenix.jdbc.PhoenixDriver
Connecting to jdbc:phoenix:hadoop01,hadoop02,hadoop03:2181
17/06/26 11:25:31 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Connected to: Phoenix (version 4.10)
Driver: PhoenixEmbeddedDriver (version 4.10)
Autocommit status: true
Transaction isolation: TRANSACTION_READ_COMMITTED
Building list of tables and columns for tab-completion (set fastconnect to true to skip)...
91/91 (100%) Done
Done
sqlline version 1.2.0
0: jdbc:phoenix:hadoop01,hadoop02,hadoop03:21>
```
## 5.使用例子 ##
- 批处理方式  
我们建立sql 名叫 us_population.sql 内容是
```
CREATE TABLE IF NOT EXISTS us_population (  state CHAR(2) NOT NULL,  city VARCHAR NOT NULL,  population BIGINT  CONSTRAINT my_pk PRIMARY KEY (state, city));
```
建立一个文件 us_population.csv  
```
NY,New York,8143197
CA,Los Angeles,3844829
IL,Chicago,2842518
TX,Houston,2016582
PA,Philadelphia,1463281
AZ,Phoenix,1461575
TX,San Antonio,1256509
CA,San Diego,1255540
TX,Dallas,1213825
CA,San Jose,912332
```
再创建一个文件 us_population_queries.sql  
```
SELECT state as "State",count(city) as "City Count",sum(population) as "Population Sum" FROM us_population GROUP BY state ORDER BY sum(population) DESC;
```
然后一起执行
```
[yangql@hadoop01 phoenix]$ python psql.py hadoop01,hadoop02,hadoop03:2181 us_population.sql us_population.sql us_population_queries.sql
```
用hbase shell 看下会发现多出来一个 US_POPULATION 表，用scan 命令查看一下这个表的数据

```
hbase(main):002:0> scan 'US_POPULATION'
ROW                                          COLUMN+CELL                                                                                                                    
 AZPhoenix                                   column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 AZPhoenix                                   column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00\x16MG                                                   
 CALos Angeles                               column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 CALos Angeles                               column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00:\xAA\xDD                                                
 CASan Diego                                 column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 CASan Diego                                 column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00\x13(t                                                   
 CASan Jose                                  column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 CASan Jose                                  column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00\x0D\xEB\xCC                                             
 ILChicago                                   column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 ILChicago                                   column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00+_\x96                                                   
 NYNew York                                  column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 NYNew York                                  column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00|A]                                                      
 PAPhiladelphia                              column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 PAPhiladelphia                              column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00\x16S\xF1                                                
 TXDallas                                    column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 TXDallas                                    column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00\x12\x85\x81                                             
 TXHouston                                   column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 TXHouston                                   column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00\x1E\xC5F                                                
 TXSan Antonio                               column=0:\x00\x00\x00\x00, timestamp=1498491908701, value=x                                                                    
 TXSan Antonio                               column=0:\x80\x0B, timestamp=1498491908701, value=\x80\x00\x00\x00\x00\x13,=                                                   
10 row(s) in 0.1780 seconds

```
- 命令行方式
```
[yangql@hadoop01 bin]$ python sqlline.py hadoop01,hadoop02,hadoop03:2181
```
退出命令行的方式是执行 !quit  
命令开头需要一个感叹号，使用help可以打印出所有命令  
hbase建立表  
```
create 'employee','company','family'
put 'employee','row1','company:name','ted'
put 'employee','row1','company:position','worker'
put 'employee','row1','family:tel','13600912345'
put 'employee','row2','company:name','michael'
put 'employee','row2','company:position','manager'
put 'employee','row2','family:tel','1894225698'
scan 'employee'
```
关于映射表在建立映射表之前要说明的是，Phoenix是大小写敏感的，并且所有命令都是大写，如果你建的表名没有用双引号括起来，那么无论你输入的是大写还是小写，建立出来的表名都是大写的，如果你需要建立出同时包含大写和小写的表名和字段名，请把表名或者字段名用双引号括起来
你可以建立读写的表或者只读的表，他们的区别如下
读写表：如果你定义的列簇不存在，会被自动建立出来，并且赋以空值
只读表：你定义的列簇必须事先存在
建立映射
```
CREATE TABLE IF NOT EXISTS "employee" ("no" CHAR(4) NOT NULL PRIMARY KEY, "company"."name" VARCHAR(30),"company"."position" VARCHAR(20), "family"."tel" CHAR(11), "family"."age" INTEGER);
```
- IF NOT EXISTS可以保证如果已经有建立过这个表，配置不会被覆盖
- 作为rowkey的字段用 PRIMARY KEY标定
- 列簇用 columnFamily.columnName 来表示
- family.age 是新增的字段，我之前建立测试数据的时候没有建立这个字段的原因是在hbase shell下无法直接写入数字型，等等我用UPSERT 命令插入数据的时候你就可以看到真正的数字型在hbase 下是如何显示的
- 查询
```
SELECT * FROM "employee";
```
- 插入/更改数据
插入或者更改数据在Phoenix里面是一个命令叫 UPSERT 意思是 update + insert
UPSERT INTO "employee" VALUES ('row3','billy','worker','16974681345',33);
```
UPSERT INTO "employee" VALUES ('row3','billy','worker','16974681345',33);
```
