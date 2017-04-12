---
title: Hive数据库创建、表创建、加载数据
date: 2017-02-28 12:54:59
tags: 数据库，表，数据加载
toc: true
categories: Hive
---
## 1.数据库 ##
### 1.1.创建数据库 ###
在Hive数据库是一个命名空间或表的集合。dbproperties,按照键值对的方式增加文档的说明。此语法声明如下：
```
CREATE DATABASE|SCHEMA [IF NOT EXISTS] <database name>
with dbproperties(key=value,key=value)
```
在这里，IF NOT EXISTS是一个可选子句，通知用户已经存在相同名称的数据库。执行创建一个名为pactera数据库：
<!-- more -->
```
CREATE DATABASE IF NOT EXISTS pactera with dbproperties('creator'='yangql','date'='2017-02-28');
```
或者
```
create schema if not EXISTS pactera;
```
查询数据库
```
hive> show databases;
OK
default
hello
pactera
Time taken: 0.083 seconds, Fetched: 3 row(s)
hive>
```
### 1.2.删除数据库 ###
删除数据库声明语法：
```
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name
[RESTRICT|CASCADE];
```
执行删除刚创建的数据库pactera
```
DROP DATABASE IF EXISTS pactera;
```
或者
级联删除数据库(当数据库还有表时，级联删除表后在删除数据库),默认是restrict
```
DROP DATABASE IF EXISTS pactera CASCADE;
```
### 1.3.查看数据库相关信息 ###
查看数据库的描述信息和文件目录位置路径信息
```
hive> desc database pactera;
```
查看数据库的描述信息和文件目录位置路径信息,加上数据库键值对的属性信息  
```
desc database extended pactera;
```
### 1.4.修改数据库 ###
只能修改数据库的键值对属性值。数据库名和数据库所在的目录位置不能修改
```
alter database pactera set dbproperties('edited-by'='yangql');
```
## 2.创建表 ##
- tblproperties：按照键值对的格式为表增加额外的文档说明，也可用来表示数据库连接的必要的元数据信息
- hive会自动增加二个表属性：last_modified_by(最后修改表的用户名)，last_modified_time(最后一次修改的时间)
### 2.1.内部表 ###
- 创建表实例
```
create table if not exists pactera.t1(name string comment 'your name',salary float comment 'salary')
comment 'this is table for test'
tblproperties('creator'='yangql','created_at'='2014-11-13 09:50:33');
```
- 查看和列举表的tblproperties属性信息
```
show tblproperties t1;
```
- 使用like在创建表的时候，拷贝表模式(而无需拷贝数据)
```
create table if not exists pactera.t2 like pactera.t1;
```
- 查看表的详细结构信息（也可以显示表是管理表，还是外部表。还有分区信息）
```
desc extended pactera.t1;
```
- 使用formatted信息更多些，可读性更强
```
desc formatted pactera.t1;
```
### 2.2.外部表 ###
删除外部表时，表的元数据会被删除掉，但是数据不会被删除。如果数据被多个工具共享，可以创建外部表。
示例
```
create external table if not exists pactera.t2(
name string comment 'name',
salary float comment 'salary')
comment 'this is a test table '
tblproperties('creator'='yangql','created_at'='2017-03-13 09:50:33')
```
### 2.3.分区表 ###

```
create table if not exists pactera.t3(
name string comment 'name',
salary float comment 'salary')
comment 'this is a test table'
partitioned by(country string,state string)
STORED AS rcfile
tblproperties('creator'='yangql','created_at'='2014-11-13 09:50:33')
```
--查看表中存在的所有分区
```
show partitions table_name;
```
--查看表中特定分区
```
show partitions table_name partition(country=’US’);
```
## 3.删除，修改表 ##
- 删除表
```
drop table if exists table_name;
```
- 修改表-表重命名
```
alter table old_table_name rename to new_table_name;
```
- 增加分区
```
alter table table_name add if not exists partition(year=2011,month=1,day=1)
```
- 修改分区存储路径
```
alter table table_name partition(year=2011,month=1,day=2)
set location ‘/logs/2011/01/02’;
```
- 删除某个分区
```
alter table table_name drop if exists partition(year=2011,month=1,day=2);
```
- 修改列信息,after:字段移到severity字段之后,first:移动到第一个位置
```
alter table table_name
change column old_name new_name int
comment 'this is comment'
after severity;
```
- 增加列
```
alter table table_name add columns(app_name string comment 'application name');
```
- 删除或者替换列
```
alter table table_name replace columns(hms int comment 'hhh');
```
- 修改表属性
```
alter table table_name set tblproperties('notes'='this is a notes');
```
- 修改存储属性
```
alter table table_name partition(year=2011,month=1,day=1) set fileformat sequencefile;
```
