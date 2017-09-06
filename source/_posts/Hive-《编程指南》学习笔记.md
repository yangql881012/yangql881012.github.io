---
title: Hive-《编程指南》学习笔记
date: 2017-04-17 08:54:59
tags: Hive
toc: true
categories: 大数据技术
---
## 基础操作 ##
1. `hive.metastore.warehouse.dir`表存储所位于的顶级文件目录，默认是/user/hive/warehouse  
2. 可以为不同的用户指定不同的目录，避免相互影响.`set hive.metastore.warehouse.dir=/user/myname/hive/warehouse`
3. hive默认数据是derby，他会在每个命令启动的当前目录创建metadata_db目录。可以设置hive-site.xml
```
javax.jdo.option.ConnectionURL=jdbc:derby:;databaseName=/home/yangql/app/hive/metastore_db;create=true
```
4. hive配置连接mysql数据库
```
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://hadoop01:3306/hive?createDatabaseIfNotExist=true</value>
    <description>
      JDBC connect string for a JDBC metastore.
      To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
      For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
    </description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
    <description>Username to use against metastore database</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>root</value>
    <description>password to use against metastore database</description>
  </property>
```
<!-- more -->

5. hive --service name 启动某个服务

|选项|名称|描述|
|---|---|---|
|cli|命令行界面|用户定义表，执行查询，默认服务|
|hiveserver|Hive Server|监听来自其他进程的Thrift连接的一个守护进程|
|hwi|hive web界面||
|jar||hadoop jar的一个扩展|
|metastore||启动一个扩展的Hive元数据服务|
|rcfilecat||一个可以打印出RCFile格式文件工具的内容|

--auxpath:选项允许用户一个以冒号分割的附属Jar包，这些文件中包含有用户可能需要的自定义扩张。   
--config 文件目录，允许用户覆盖$HIVE_HOME/conf中默认的属性配置，而指向一个新的配置文件目录。

6. hive -h 显示帮助信息
```
yangql@hadoop01 conf]$ hive -h
which: no hbase in (/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/opt/jdk1.8.0_91/bin:/opt/scala-2.12.1/bin:/home/yangql/bin:/home/yangql/app/hadoop-2.7.2/bin:/home/yangql/app/sqoop-1.4.6/bin:/home/yangql/app/spark-2.1.0-bin-hadoop2.7/bin:/home/yangql/app/zookeeper-3.4.6/bin:/home/yangql/app/hive-2.1.1/bin:/home/yangql/app/kafka-2.12/bin:/home/yangql/app/flume-1.7.0/bin)
Unrecognized option: -h
usage: hive
 -d,--define <key=value>          Variable subsitution to apply to hive
                                  commands. e.g. -d A=B or --define A=B
    --database <databasename>     Specify the database to use
 -e <quoted-query-string>         SQL from command line
 -f <filename>                    SQL from files
 -H,--help                        Print help information
    --hiveconf <property=value>   Use value for given property
    --hivevar <key=value>         Variable subsitution to apply to hive
                                  commands. e.g. --hivevar A=B
 -i <filename>                    Initialization SQL file
 -S,--silent                      Silent mode in interactive shell
 -v,--verbose                     Verbose mode (echo executed SQL to the
                                  console)
```
7. hive中变量和属性命名空间
- hivevar 用户自定义变量
- hiveconf hive相关的配置文件
- system Java定义的配置属性
- env shell环境变量
- 定义一个变量`hive --define foo=bar;`在cli中，变量时先被替换掉，才提交给查询处理器
```
hive> set hivevar:foo;
hivevar:foo=bar
hive> create table test(id INT,${hivevar:foo} string);
OK
Time taken: 1.769 seconds
hive> desc test;
OK
id                  	int                 	                    
bar                 	string              	                    
Time taken: 0.238 seconds, Fetched: 2 row(s)
hive>
```
- hiveconf配置hive行为的所有属性，例如配置答应出当前数据的名字
```
[yangql@hadoop01 conf]$ hive --hiveconf hive.cli.print.current.db=true
which: no hbase in (/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/opt/jdk1.8.0_91/bin:/opt/scala-2.12.1/bin:/home/yangql/bin:/home/yangql/app/hadoop-2.7.2/bin:/home/yangql/app/sqoop-1.4.6/bin:/home/yangql/app/spark-2.1.0-bin-hadoop2.7/bin:/home/yangql/app/zookeeper-3.4.6/bin:/home/yangql/app/hive-2.1.1/bin:/home/yangql/app/kafka-2.12/bin:/home/yangql/app/flume-1.7.0/bin)

Logging initialized using configuration in file:/home/yangql/app/hive-2.1.1/conf/hive-log4j2.properties Async: true
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. tez, spark) or using Hive 1.X releases.
hive (default)> set hiveconf:hive.cli.print.current.db
              > ;
hiveconf:hive.cli.print.current.db=true
hive (default)>
```
8. 一次使用命令  
```
[yangql@hadoop01 conf]$ hive -e "select * from t2";
-S:静默方法
//将查询结果输出到文件中
[yangql@hadoop01 bin]$ hive -S -e "select * from t3 limit 3" > test.txt
```
9. 查询变量的方法
```
[yangql@hadoop01 bin]$ hive -S -e "set" | grep warehouse
which: no hbase in (/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/opt/jdk1.8.0_91/bin:/opt/scala-2.12.1/bin:/home/yangql/bin:/home/yangql/app/hadoop-2.7.2/bin:/home/yangql/app/sqoop-1.4.6/bin:/home/yangql/app/spark-2.1.0-bin-hadoop2.7/bin:/home/yangql/app/zookeeper-3.4.6/bin:/home/yangql/app/hive-2.1.1/bin:/home/yangql/app/kafka-2.12/bin:/home/yangql/app/flume-1.7.0/bin)
hive.metastore.warehouse.dir=/user/hive/warehouse
hive.warehouse.subdir.inherit.perms=true
[yangql@hadoop01 bin]$
```
9. 从文件执行查询
```
[yangql@hadoop01 bin]$ hive -f querysql.hql
可以进入后，再用source命令：
hive> source querysql.hql
```
10. hive构建src表，相当于关系数据库里dual表
```
create table src(s STRING)
echo "one row">/home/yangql/myfile.txt
[yangql@hadoop01 bin]$ hive -e "LOAD DATA LOCAL INPATH '/home/yangql/myfile.txt' INTO TABLE SRC";
```
11. hivesrc文件
Hive自动在HOME目录下寻找名为.hiverc的文件，在提示符出现之前执行这个文件.
```
set hive.cli.print.current.db=true
```
12. 历史命令,hive 会将最近10000条行命令记录到文件$HOME/.hivehistory中
13. hive 内使用shell命令 ! command ;
14. hive 内使用hadoop命令 dfs -ls
15. 开启打印字段名称：hive.cli.print.header=true
16. 基本数据类型    

|数据类型|长度|
|---|---|
|TINYINT|1BYTE|
|SMALLINT|2BYTE|
|INT|4BYTE|
|BIGINT|8BYTE|
|BOOLEAN|TRUE FALSE|
|FLOAT|单精度|
|DOUBLE|双精度|
|STRING|字符串|
|TIMESTAMP|整数，浮点数，字符串|
|BINARY|字节数组|
17. 集合数据类型有三种，STRUCT('john','Doe')，MAP('first','John','last','Doe')，ARRAY('John','Doe'),使用例子
```
create table employee(
name STRING,
salary FLOAT,
subemloyers ARRAY<STRING>,
deducation MAP<STRING,FLOAT>,
address STRUCT<street:STRING,citr:STRING,state:STRING>
)
```
18. hive中默认的记录和字段分隔符

|分隔符|描述|
|---|---|
|\n|记录分隔符|
|^A CTRL-A|列分隔符，在create table中八进制\001表示|
|^B|STRUCT，ARRAY,MAP分割，在create table中八进制\002表示|
|^C|MAP键值对分割，在create table中八进制\003表示|
19. hive 读时模式，在加载数据的时候，并不去验证数据与模式是否匹配，而是在查询时进行（schema on read）
## 数据定义 ##
1. hive不支持行级插入，更新，删除操作，也不支持事务
2. hive中的数据库本质上仅仅只是一个目录或者命名空间，如果没有显示指定，默认数据库default,创建数据库如下(if not exists可选).hive会给每个数据库创建一个目录,default除外。也可以通过location
指定目录
```
--创建
hive (default)> create database if not exists financials;
--查看
hive (default)> show databases;
--加正则表达式查看
hive (default)> show databases like 'pactera*';
--指定目录
hive (default)> create database specialdir location '/user/hive/warehouse';
--添加注释
hive (default)> create database  financials01 comment 'test';
--查看数据库注释
hive (default)> desc database financials01
--为数据库增加键值对信息
hive (default)> create database financialextend with dbproperties('createor'='yangql','date'='20170414');
--显示扩展信息
hive (default)> desc database extended financialextend;
```
3. 切换数据库`use databaseName`
4. 删除数据库 drop database if exists databaseName ,if exists是可选的。默认情况下，hive不允许删除一个包含表的数据库。有两种方式
- 先删除数据库下所有表，再删除数据库
- 加上casecase关键字。hive会将数据库下的所有表删除，然后删除数据库
5. 修改数据库，修改数据库只能修改dbproperties，数据库其它元数据信息不能修改。
```
hive (default)> alter database pactera set dbproperties('edited_by','Joe');
```
6. 创建表
```
create table employees(
name STRING comment 'employee Name',
salary FLOAT comment 'salary'
)
comment 'desc of table'
tblproperties('creator'='me','created_at'='2017-04-15');
```
7. 拷贝表模式
```
create table if not exists employees2 like employees;
```
8. show tables:显示当前数据库下所有表，show tables in pactera:显示某个数据库下所有表
9. 查看表的扩展信息
```
desc extended employees; --显示的信息多
desc formatted employees; --显示的信息可读性好
```
10. 管理表，也称内部表，hive会控制着数据的生命周期，当我们删除一个表时，hive也会删除表对应的数据。
11. 外部表，数据被多个工具共享时，可创建外部表，删除表后，数据不会被删除，删除的只是表的元数据，创建外部表如下:
```
create external table if not exists stocks(
stock_id STRING,
price FLOAT
)
row format delimited
fields terminated by ','
location '/user/hive/warehouse/stocks';
```
12. 分区表该表了hive对数据存储的组织方式，创建分区表
```
create table employees(
name string,
salary float,
address struct<street:string,city:string,state:string,zip:int>
)
partitioned by (country string,state string);

--查看分区的信息
show partitions employees;
--查看某个指定的分区
show partitions employees partition(country='US',state='AK')
--设置参数 hive.mapred.mode=strict
set hive.mapred.mode=strict=true
hive (mydb)> set hive.mapred.mode=strict;
hive (mydb)> set hive.mapred.mode
           > ;
hive.mapred.mode=strict
hive (mydb)> select * from employees;
FAILED: SemanticException Queries against partitioned tables without a partition filter are disabled for safety reasons. If you know what you are doing, please make sure that hive.strict.checks.large.query is set to false and that hive.mapred.mode is not set to 'strict' to enable them. No partition predicate for Alias "employees" Table "employees"
hive (mydb)>
```
将hive设置为strict后，对分区进行查询，如果没有指定where语句，将会禁止提交这个任务。

13. 自定义表的存储格式
```
create table employee02(
name string,
salary float
)
row format delimited
fields terminated by '\001'
collection items terminated by '\002'
map keys terminated by '\003'
lines terminated by '\n'
stored as textfile;
```
row format delimited
fields terminated by '\001' --列分割
collection items terminated by '\002' --集合分割
map keys terminated by '\003' --map 分割
lines terminated by '\n' --行分割
stored as textfile; --存储格式

14. `alter table `修改表，修改的只是元数据，并不会修改数据本身。
```
--重命名表名
hive (mydb)> alter table employee02 rename to employee03;
--增加分区
alter table log_message add if not exists
partition(year=2017,month=1,day=1) location '/logs/2017/01/01'
partition(year=2017,month=1,day=2) location '/logs/2017/01/02'
--修改分区的路径
alter table log_message partition(year=2017,month=1,day=1) set location '/logs/2017/01/01'
--删除分区
alter table log_message drop if exists partition(year=2017,month=1,day=1)
--修改字段的列,修改列名，移动到某个字段之后，fisrt移动到最前
alter table employee03 change column salary salary_total float comment 'total salary' after name;
alter table employee03 change column salary_total total_salary double comment 'total salary' first;
--增加列
alter table employee03 add columns(
app_name string,
session_id string
)
--移除替换列
alter table employee03 replace columns(
name string,
salary double,
app_name string,
session_id string
);
--修改表的属性
alter table employee03 set tblproperties('notes'='the process id is no longer captured')
--表中文件被改，增加钩子
hive -e "alter table log_message touch partition(year=2012,moonth=1,day=1);"
--将分区内文件打成一个hadoop压缩包，HAR.降低文件系统中的文件数以及减轻namenode压力，而不会减少任何的存储空间,反向操作,unarchive
alter table log_message archive partition(year=2012,month=01,day=01)
--防止分区被删除
alter table log_message partition(year=2012,month=01,day=01) enable no_drop;
--禁止查询分区数据
alter table log_message partition(year=2012,month=01,day=01) enable offline;
```
## 数据操作 ##
1. 将数据加载到表中，如果目录不存在的话，会先创建目录，然后再将数据拷贝到目录下.通常指定的路径是一个目录，而不是单个文件.这使得用户可以组织数据到多文件中,同时可以在不修改hive脚本的前提下修改文件名
```
load data loca inpath '${env:HOME}/employee' overwrite into table employees partition(country='US',statue='CA')

```
2. load data local inpath:拷贝本地数据到位于分布式文件系统上的目标位置
3. load data inpath:转移数据到目标位置(转移)，要求源文件，目标文件，目录应该在同一个集群中。
4. overwrite:目标文件夹之前存在的文件会先被删除。如果没有使用overwrite关键字，而目标文件下已经存在相同文件名时，会保留之前的文件并重命名问，文件名_序号
5. inpath目录下不能再包含目录
6. hive并不会验证用户装载的数据和表的模式是否匹配，但是会验证文件格式是否与表定义的是否一直。如表定义时定义的存储格式是sequencefile,那么装载进去的文件也应该是sequencefile才对。
7. 通过查询语句向表中插入数据，适用场景：目标数据的文件格式，分隔符与当前需要的数据格式，分隔符不想同时，可以使用这种方式。其中overwrite表示覆盖，into追加
```
insert overwrite table employees
partition(country='US',state='OR')
select * from staged_employee se where se.cnty='US' and se.st='OR';
--一次查询，多次插入
from staged_employee se
insert overwrite table employees
partition(country='US',state='OR')
select * from staged_employee se where se.cnty='US' and se.st='OR'
insert overwrite table employees
partition(country='US',state='CA')
select * from staged_employee se where se.cnty='US' and se.st='CA'
```
8. 动态分区插入,hive会根据最后两列来确认分区字段country,state的值，源表字段值和输出分区值之间的关系是根据位置而不是根据命名来的。用户也可以混合使用动态分区和静态分区,但是静态分区必须在动态分区之前
```
insert overwrite table employees
partition(country,state)
select ...,cnty,st from staged_employee;
--混合使用动态分区和静态分区
insert overwrite table employees
partition(country='US',state)
select ...,cnty,st from staged_employee where cnty='US';
```
9. 动态分区的属性
```
--开启和关闭动态分区
hive.exec.dynamic.partition=true
--设置成nostrict，表示允许所有分区都是动态的
hive.exec.dynamic.partition.mode=nostrict
--每个node创建的最大分区数
hive.exec.max.dynamic.partitions.pernode=100
--一个动态分区创建语句可以创建的最大分区数
hive.exec.max.dynamic.partitions=1000
--全局可以创建的最大文件数
hive.exec.max.created.files=100000
```
10. 单个查询语句中创建表并加载数据，适用场景：从一个大表中选取部分需要的数据
```
create table ca_employee as select name,salary,address from employees where state='CA';
```
11. 导出数据，如果文件的格式满足用户需求，直接将文件拷贝到本地就行
```
hdfs dfs -cp source_path target_path
```
如果不满足需求
```
insert overwrite local directory '/home/yangql/data/employees' select * from user_info;
```
可以指定多个输出文件夹目录
```
from staged_employee se
insert overwrite directory 'directory1' select * from employees where name='1'
insert overwrite directory 'directory2' select * from employees where name='2'
```
12. 导出数据如果不满足用户的格式时，可以先创建一个临时表，将数据插入到临时表中，再从临时表中导出。
## 查询 ##
1. 查询字段是集合时，会以JSON格式显示。取集合数据时，使用.ARRAY[0],Map时。MAP[key],STRUCT时。STRUCT.name
2. 使用正则表达式指定列
3. 表生成函数：explode
4. 什么情况下hive可以避免进行Mapreduce
- 本地查询select * from tableName,Hive读取对应存储目录下的文件
- where条件只有分区字段的情况
- set hive.exec.mode.local.auto=true;Hive会尝试使用本地模式执行其它的操作
5. RLIKE使用正则表达式
6. 其它SQL方言中in exists可以使用left semi join（左半开连接）实现，但是不能查询右边表的字段。对于左表中的一条记录，在右边表中一旦找到匹配记录，就会停止扫描
```
select t1.* from user_info t1
left semi join ids t2
on t1.user_ids=t2.user_ids
```
7. map-side join ,所有表中只有一张小表时，那么可以在最大的表通过mapper的时候将小表完全放到内存中，0.7以前通过`/*+ MAPJOIN(表别名)  */`，0.7后，设置参数
```
set hive.auto.convert.join=true
--设置可以配置使用这个优化的小表的大小
set hive.mapjoin.smalltable.filesize=25000000
```
right outer join,full outer join 不支持这个优化
8. order by 全局排序，sorted by 局部排序
9. distribute by:默认情况下，Mapreduce计算框架会依据map输入的键计算相应的哈希值，然后按照得到哈希键值对均匀分发到多个reducer中去，这也意味着，当我们使用sort by时，不同的reducer的输出内容会有
明显的重叠。如果我们希望同一类型的交易数据在同一个 reducer处理。我们可以使用`distribute by` ，如将同一性别的用户放到同一reducer上处理,distribute by 必须在sort by之前
```
select * from user_info
distribute by sex
sort by sex;
```
10. cluster by :相当于 distribute by sex , sort by sex ,两个语句中涉及到的列完全相同，采用的是升序排列。那cluster by 将相当于是这两个语句的简写
```
select * from user_info
cluster by sex;
```
12. 类型转换 cast(value as type),将value转换为相应的数据类型。当不能转换时，返回NULL值。将浮点数转为整数round,floor,ceil。
13. 抽样查询， tablesample(bucket 3 out of 10 on rand()) s。分母10表示数据将会被散列的桶的个数，分子表示将会选择的桶的个数。
```
--如果按照rand()函数进行随机抽样，这个函数会返回一个随机数
select * from user_info tablesample(bucket 3 out of 10 on rand()) s;
--按照指定的列进行随机抽样，同一语句返回的结果是一样的。
select * from user_info tablesample(bucket 3 out of 10 on user_ids) s;
```
14. 数据块抽样,按照抽样百分比进行抽样。这种是基于行数的。按照数据路径下的数据块百分比进行抽样。这种抽样的最小单元是HDFS的一个数据块。如果数据的大小小于普通块大小128MB。那么将会返回所有行。
```
select * from user_info tablesample(0.1 percent) s;
```
基于百分比的抽样方式提供的变量。用于控制基于数据块的调优的种子信息。
```
<property>
    <name>hive.sample.seednumber</name>
    <value>0</value>
    <description>A number used to percentage sampling. By changing this number, user will change the subsets of data sampled.</description>
</property>
```
15. 分桶表的数据裁剪,数据将会被聚集成10个buckets
```
create table user_info_ids(user_ids string) clustered by (user_ids) into 10 buckets;
insert overwrite table user_info_ids select user_ids from user_info;
```
## 视图 ##
1. 创建视图,创建Hive视图的两个作用1.降低查询语句的复杂度，2.隐藏敏感信息。Hive先解析视图，然后再使用解析结果再来解析整个查询语句
```
create view user_info_female as select * from user_info where sex='female';
```
2. 删除视图:drop view if exists xxx

## 索引 ##
1. 创建索引
```
create index user_info_index1 on table user_info(sex)
as 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler'
with deferred rebuild
idxproperties('creator'='me')
in table user_info_index
comment 'index';
```
2. 创建bitmap索引
```
create index user_info_index2 on table user_info(sex)
as 'BITMAP'
with deferred rebuild
idxproperties('creator'='me')
in table user_info_index2
comment 'index';
```
3. 重建索引：如果用户指定了deferred rebuild,那么新索引将呈现空白状态，在任何时候，都可以进行第一次索引创建或者使用alter index对索引进行重建
```
alter index user_info_index2 on user_info rebuild;
```
4. 显示索引
```
show formatted index on user_info;
```
5. 删除索引
```
drop index if exists user_info_index2 on user_info;
```
## 调优 ##
1. explain如何将查询转化为Mapreduce
2. 限制调整，在多数情况下，limit语句还是要执行所有语句，然后再返回部分结果，可以配置hive属性，当使用limit时，对源数据进行抽样。
```
<property>
	<name>hive.limit.optimize.enable</name>
	<value>true</value>
</property>
```
其它，`hive.limit.row.max.size` `hive.limit.optimize.limit.file`两个参数
3. Join优化：将大表放在JOIN最右边，或者直接使用/* streamtable(table_name)*/指定。如果一个表中很小，可以完整载入到内存中，此时Hive可以执行一个map-side JOIN.减少reduce的过程。
4. 启动本地模式，设置属性`hive.exec.model.local.auto=true`启动本地模式。（有时候当Hive输入数据量很小时，为查询触发执行任务的时间消耗可能比实际执行job的时间还多）。如果要设置所有用户使用这个配置，
可以在hive-site.xml文件中进行配置
```
<property>
	<name>hive.exec.model.local.auto</name>
	<value>true</value>
</property>
```
5. 并行执行：Hive会将一个查询转化为一个或多个阶段。这样的阶段可以是Mapreduce阶段、抽样阶段、合并阶段、limit阶段，或者Hive执行阶段过程中可能需要的其他阶段。默认情况下，Hive每次只能执行一个阶段。
但是有些阶段是可以并行执行，这样就可能使得整个Job的执行时间缩短。设置参数hive.exec.parallel,开启并发执行
```
<property>
	<name>hive.exec.parallel</name>
	<value>true</value>
</property>
```
6. 严格模式：防止用户执行那些可能产生不好影响查询。可以设置`hive.mapred.mode` 值为strict
```
set hive.mapred.mode=strict
```
可以禁止3种类型的查询
- 对于分区表，除非where语句中含有分区字段的过滤条件来限制数据范围，否则不允许执行，用户不允许扫描所有分区的数据。通常分区表都有非常大的数据集，且增长迅速
- 对于使用了order by 子句的查询，要求必须使用limit语句。因为order by子句为了执行排序过程会将所有数据分发到同一个reducer中处理，强制要求用户增加limit防止reducer额外执行很长一段时间
- 限制笛卡尔积的查询，两个表关联条件写在where中时，Hive并不会执行优化
7. 调整mapper和reducer个数：Hive通过查询划分为一个或多个Mapreduce任务达到并行的目的。确定最佳的mapper个数和reducer个数取决于多个变量，数据量的大小，对数据执行的操作类型等。
过多的mapper和reducer任务，就会导致启动阶段，调度和运行Job过程中产生过多的开销。而数量设置过少，就没有充分的利用好集群内在的并行性。`hive.exec.reducers.bytes.per.reducer`默认是1G，可以修改其大小。
如果只是根据输入量的大小，可能导致reducer不准确，当mapper后数据量多，reducer不够用，mapper数据量小，reducer过多。我们可以设置参数限制reducer个数,hive默认的reducer个数是3
```
mapred.reduce.tasks
```
当在集群上处理大任务时，为了控制资源的利用情况，属性`hive.exec.reducers.max`可以设置reducer的最大值。一个Hadoop集群可以提供的mapper和reducer资源个数（插槽）是固定的。某个大Job可能会消耗完所有的插槽，从而导致
其它的Job无法执行，通过设置属性`hive.exec.reducers.max`可以阻止某个查询消耗太多的reducer资源。建议大小=（集群总Reducer槽位个数 * 1.5）
8. JVM重用：适用于小文件场景和task特别多的场景。Hadoop默认的配置是使用派生的JVM来执行map和reduce任务。这是JVM的启动过程可能会造成很大的开销。尤其是执行的Job包含成百上千个task时，JVM重用可以使得
JVM实例在同一个JOB中重用N次。N值可以在 hive-site.xml中配置。
<property>
	<name>mapred.job.reuse.jvm.num.tasks</name>
	<value>10</value>
</property>
此功能的缺点时，JVM会一直占用适用到的task插槽，以便进行重用，知道任务完成后才能释放。
9. 索引：索引可以加快含有Group By语句的查询计算速度。
10. 动态分区的调整：开启动态分区严格模式时，必须保证至少有一个分区是静态的，限制查询创建最大分区数
```
--分区模式
<property>
	<name>hive.exec.dynamic.partition.mode</name>
	<value>strict</value>
</property>
-- 最大分区数
<property>
	<name>hive.exec.max.dynamic.partitions</name>
	<value>300000</value>
</property>
-- datanode上可以一次打开的文件个数
<property>
	<name>hive.exec.max.dynamic.partitions.pernode</name>
	<value>10000</value>
</property>
```
11. 虚拟列：Hive提供了两种虚拟列，一种用于将要进行划分的输入文件名，一种用于文件中的块内偏移量。当Hive产生了非预期或null的返回结果时，可以通过这些虚拟列查询
```
set hive.exec.rowoffset=true;
select input_file_name,block_offset_inside_file,line from hive_text where line like '%hive%' limit 2;
```
第三种虚拟列提供了文件的行偏移量。
```
<property>
	<name>hive.exec.rowoffset</name>
	<value>true</value>
</property>
```
## 文件格式和压缩方法 ##
1. 查询hadoop编解码器
```
hive -e "set io.compression.codecs"
```
2. 使用压缩的优势：最小化磁盘所需要的空间。减少磁盘和网络的IO操作，不过文件的压缩和解压过程会增加CPU开销
3. 开启中间数据的压缩：对中间数据进行压缩可以减少job中map和reduce task间的数据传输量。开启中间数据压缩的配置
```
<property>
 <name>hive.exec.compress.intermediate</name>
 <value>true</value>
</property>
```
4. 输出压缩
```
<property>
    <name>hive.exec.compress.output</name>
    <value>false</value>
    <description>
      This controls whether the final outputs of a query (to a local/HDFS file or a Hive table) is compressed.
      The compression codec and other options are determined from Hadoop config variables mapred.output.compress*
    </description>
  </property>
```
5. sequence file存储格式
压缩文件可以节约存储空间，但是通常这些文件是不能分割的。sequence file存储格式可以将一个文件划分为多个块，然后采用一种分割的方式对块进行压缩。如果要在hive中使用sequence file存储格式，
在create table 中通过 `stored as sequencefile`
```
create table sequence_file_tb stored as sequencefile;
```
sequence file提供了三种压缩方式，None，RECORD，BLOCK，其中RECORD级别是默认的。BLOCK级别的压缩性能最好且是可以分割的。可以在hive-site.xml中定义
```
<property>
    <name>mapred.output.compression.type</name>
    <value>BLOCK</value>
  </property>
```
6. 压缩实战
```
hive (mydb)> select * from user_info;
--开启中间压缩
hive (mydb)>set hive.exec.compress.intermediate=true
hive (mydb)> select * from user_info;--时间变少
hive (mydb)>create table user_info_0417 as select * from user_info;--输出结果没有压缩
hive (mydb)>  set mapred.map.output.compression.codec=org.apache.hadoop.io.compress.GZipCodec;--设置压缩类型
--开启输出结果压缩
hive (mydb)> set hive.exec.compress.output=true;
--输出的文件已经被压缩
create table user_info_041702 as select * from user_info;

hive (mydb)> set hive.exec.compress.output=true;
set mapred.output.compression.codec=org.apache.hadoop.io.compress.GzipCodec;
create table user_info_041703 as select * from user_info;
```
7. 存档分区：Hadoop中有一种存储格式为HAR，HAR文件内部可以有文件和文件夹。HAR文件可以减少NameNode消耗较大的代价来管理这些文件。Har访问效率低，Har文件没有被压缩，因此也不会节约存储空间。
```
--创建分区
hive (mydb)> create table hive_text(line STRING) partitioned by (folder STRING);
--增加分区
hive (mydb)> alter table hive_text add partition(folder='docs');
--加载数据
load data local inpath '${env:HIVE_HOME}/README.txt' into table hive_text partition(folder='docs');
hive (mydb)> load data local inpath '${env:HIVE_HOME}/RELEASE_NOTES.txt' into table hive_text partition(folder='docs');
--将表转换为一个归档表
set hive.archive.enabled=true;
hive (mydb)> alter table hive_text archive partition(folder='docs');
--重新将har文件提取出来
alter table hive_text unarchive partition(folder='docs')
```
## 开发 ##
1. hive-log4j2.properties：CLI和其它本地执行组件的日志
2. hive-exec-log4j2.properties：控制Mapreduce task内日志

## 函数 ##
1.  show functions; 显示当前加载的函数
2.  desc function year;查看函数的帮助文档
3. desc function extended year;：查看函数的详情
4. 函数的分类
- 标准函数UDF
- 聚合函数UDAF
- 表生成函数UDTF,表生成函数接受零个或多个输入，产生多列或多行输出
- array函数就是将一列输入转化为一个数组输出
```
hive (mydb)> select array(1,2,3) from user_info limit 1;
OK
c0
[1,2,3]
```
- explode:以array输入，然后对数组中的数据进行迭代，返回多行结果，一行一个数组元素值.explode无法产生其它的列
```
hive (mydb)> select explode(array(1,2,3)) from user_info limit 3;
OK
col
1
2
3
Time taken: 0.423 seconds, Fetched: 3 row(s)
```
```
select user_ids,sub from user_info
lateral view explode(array(1,2,3)) subView as sub
limit 3
```
5. 创建一个自定义函数UDF
- 实现UDF类
- 打包成Jar文件
- 加载JAR文件：ADD JAR /home/yangql/zodiac.jar
- create temporary function zodiac as 'org.com.yangql.KKKKK'
如果用户需要频繁的使用自定义函数。需要将相关语句加入到 `.hiverc`文件中
6. 删除UDF drop temporary function if exists zodiac;

## Hive文件及记录格式 ##
1. 从表中读取数据时，Hive会使用InputFormat，向表中写入数据时，使用OutputFormat
2. 文本文件可以与其它的工具功效数据，但是存储空间大。二进制文件可以节约存储空间。也可以提高I/O性能。
3. sequence file:stored as sequencefile
4. RCFILE:列式存储
5. SerDe序列化与反序列化
6. xpath访问xml文件
```
hive (mydb)> select xpath('<a><b id="foo">b1</b><b id="bar">b2</b></a>','//@id') from src;
OK
c0
["foo","bar"]
hive (mydb)> select xpath_double('<a><b>1</b><c>2</c></a>','a/b+a/c') from src;
OK
c0
3.0
```

## Thrift服务 ##
1. Hive具有一个可选的组件叫做HiveServer或者HiveThrift，允许通过端口访问Hive，用于跨语言的服务无开发
2. 启动thriftserver:`hive --service hiveserver2`

## Hive中的权限 ##
1. 开启授权模式
```
<property>
    <name>hive.security.authorization.enabled</name>
    <value>true</value>
    <description>enable or disable the Hive client authorization</description>
</property>
<property>
    <name>hive.security.authorization.createtable.owner.grants</name>
    <value>all<value/>
    <description>
      The privileges automatically granted to the owner whenever a table gets created.
      An example like "select,drop" will grant select and drop privilege to the owner
      of the table. Note that the default gives the creator of a table no access to the
      table (but see HIVE-8067).
    </description>
</property>
```
2. 对用户授权，用户就是操作系统的用户
- 确认名字：set system:user.name
- 授权：
- 查看： show grant user yangql;
3. 对组授权
4. 对role授权
