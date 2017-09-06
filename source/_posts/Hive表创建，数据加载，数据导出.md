---
title: Hive表创建，数据加载，数据导出
date: 2017-06-15 12:54:59
tags: Hive
toc: true
categories: 大数据技术
---
hive不支持用insert语句一条一条的进行插入操作，也不支持update操作。数据是以load的方式加载到建立好的表中。数据一旦导入就不可以修改。
## 1.创建表的三种方式 ##
- 创建表的基本语法

```
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [COMMENT col_comment], ...)]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format]
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)

CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
  LIKE existing_table_or_view_name
  [LOCATION hdfs_path];

data_type
  : primitive_type
  | array_type
  | map_type
  | struct_type
  | union_type  -- (Note: Available in Hive 0.7.0 and later)

primitive_type
  : TINYINT
  | SMALLINT
  | INT
  | BIGINT
  | BOOLEAN
  | FLOAT
  | DOUBLE
  | STRING
  | BINARY      -- (Note: Available in Hive 0.8.0 and later)
  | TIMESTAMP   -- (Note: Available in Hive 0.8.0 and later)
  | DECIMAL     -- (Note: Available in Hive 0.11.0 and later)
  | DECIMAL(precision, scale)  -- (Note: Available in Hive 0.13.0 and later)
  | DATE        -- (Note: Available in Hive 0.12.0 and later)
  | VARCHAR     -- (Note: Available in Hive 0.12.0 and later)
  | CHAR        -- (Note: Available in Hive 0.13.0 and later)

array_type
  : ARRAY < data_type >

map_type
  : MAP < primitive_type, data_type >

struct_type
  : STRUCT < col_name : data_type [COMMENT col_comment], ...>

union_type
   : UNIONTYPE < data_type, data_type, ... >  -- (Note: Available in Hive 0.7.0 and later)

row_format
  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
        [NULL DEFINED AS char]   -- (Note: Available in Hive 0.13 and later)
  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]

file_format:
  : SEQUENCEFILE
  | TEXTFILE    -- (Default, depending on hive.default.fileformat configuration)
  | RCFILE      -- (Note: Available in Hive 0.6.0 and later)
  | ORC         -- (Note: Available in Hive 0.11.0 and later)
  | PARQUET     -- (Note: Available in Hive 0.13.0 and later)
  | AVRO        -- (Note: Available in Hive 0.14.0 and later)
  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname
```
- 第一种方式
```
CREATE TABLE IF NOT EXISTS crm.weblog(
ip string ,
time string
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' ;
```
- 第二种方式（包含原表数据）
```
CREATE TABLE crm.weblog_comm
AS select ip, time, req_url from crm.weblog;
```
- 第三种方式(只含表结构)
```
CREATE TABLE IF NOT EXISTS crm.weblogbak LIKE crm.weblog ;  
```

## 2.向数据表内加载文件 ##
```
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]
```
Load 操作只是单纯的复制/移动操作，将数据文件移动到 Hive 表对应的位置。
- LOCAL：本地模式
- OVERWRITE：目标表（或者分区）中的内容（如果有）会被删除，然后再将 filepath 指向的文件/目录中的内容添加到表/分区中
filepath有以下几种类型
- 相对路径，例如： 20170524/crm_cus_com.txt
- 绝对路径，例如： /user/hive/20170524/crm_cus_com.txt
- 包含模式的完整 URI，例如：hdfs://namenode:9000/user/hive/20170524/crm_cus_com.txt
```
hive> LOAD DATA LOCAL INPATH '/home/yangql/data/20170524/crm_cus_com.txt' OVERWRITE INTO TABLE crm.crm_cus_com;
```

## 3.将查询结果插入Hive表 ##
- 基本模式
```
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement
--例子
hive (crm)> insert overwrite table crm_cus_com0615 select * from crm_cus_com;
```
- 多插入模式
```
FROM from_statement
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1
[INSERT OVERWRITE TABLE tablename2 [PARTITION ...] select_statement2] ...
--例子
from crm_cus_com
insert overwrite table crm_cus_com0615
select *
insert overwrite table crm_cus_com0616
select *
;
```
- 追加模式,不删除原来的数据
```
INSERT INTO  TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement

--例子
hive (crm)> insert into  table crm_cus_com0615 select * from crm_cus_com;
```
- 自动分区模式
```
INSERT OVERWRITE TABLE tablename PARTITION (partcol1[=val1], partcol2[=val2] ...) select_statement FROM from_statement
```

## 4.将查询结果写入HDFS文件系统 ##
数据写入文件系统时进行文本序列化，且每列用^A 来区分，\n换行

```
INSERT OVERWRITE [LOCAL] DIRECTORY directory1
SELECT ... FROM ...

FROM from_statement
INSERT OVERWRITE [LOCAL] DIRECTORY directory1 select_statement1
[INSERT OVERWRITE [LOCAL] DIRECTORY directory2 select_statement2]

--例子
hive (crm)> insert overwrite local directory '/home/yangql/data/test' select * from crm_cus_com;
```
