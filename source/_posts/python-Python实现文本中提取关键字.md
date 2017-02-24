---
title: Python实现文本中提取关键字
date: 2017-02-16 10:54:59
tags: Python
categories: Python
---
需求描述：
1.提供需要统计的表清单  
2.将存储过程中使用的表统计出来  
3.将结果插入到数据库中  
<!-- more -->
关键代码如下：
```
#_*_ coding:utf-8 _*_
#!/usr/bin/env python
#==========================================================
# ScriptName : findTables
# Author     : Yangql
# Created    : 2015年3月12日 上午11:16:34
# CopyRight  : (C) Ron Yang 2017
# Licence    :
# Purpose    :
#==========================================================
import  os
import cx_Oracle
import re

print("cx_Oracle.version:", cx_Oracle.version)
host = "127.0.0.1"
port = "1521"
sid = "orcl"
dsn = cx_Oracle.makedsn(host, port, sid)
connection = cx_Oracle.connect("pds", "pds", dsn)
cursor_tbs = cx_Oracle.Cursor(connection)  # 返回连接的游标对象
cursor_src = cx_Oracle.Cursor(connection)  # 返回连接的游标对象
cursor_inser = cx_Oracle.Cursor(connection)
#sql_tbs = "select t.table_name from pds_tables t "
#sql_src = "select t.name,t.TEXT from user_source t where t.TYPE='PROCEDURE' "
sql_tbs = "select t.table_name from pds_tables_b t "
sql_src = "select t.name,t.TEXT from user_source t where t.TYPE='PROCEDURE' and (t.name like 'UP_B%' or t.name like 'P_B%' or t.name like 'B%')"
str_tbs = cursor_tbs.execute(sql_tbs).fetchall()
str_src = cursor_src.execute(sql_src).fetchall()
#print(len(cursor_src.fetchall()))
for row_tbs in str_tbs:
    #print(str(row_tbs[0]).strip().upper()) # get the first element of tuple
    for row_src in str_src:
        #print(row_src)
        #print(row_src[0].strip().upper())
        #print(row_src[1].strip().upper())

        table_names=re.findall(row_tbs[0].strip().upper(),row_src[1].strip().upper())
        table_name=""
        if table_names:
            #print(table_names[0])
            table_name=table_names[0]
        proc_name=row_src[0].strip().upper()
        #将结果插入到数据库
        #print(" insert into pds_tables_procs(PROC_NAMES,TABLE_NAME) VALUES ('" +proc_name +"','" +table_name+ "')")
        if table_name and proc_name:
            insert_sql = " insert into pds_tables_procs_b(PROC_NAMES,TABLE_NAME) VALUES ('" + proc_name + "','" + table_name + "')"
            cursor_inser.execute(insert_sql)
            connection.commit()
#关闭 cursor connection
cursor_tbs.close()
cursor_src.close()
cursor_inser.close()
connection.close()
print("statistics finished!!!")

```
