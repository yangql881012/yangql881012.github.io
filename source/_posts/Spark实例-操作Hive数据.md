---
title: Spark实例-操作Hive数据
date: 2017-05-07 10:54:59
tags: Spark
toc: true
categories: 大数据技术
---
Spark操作Hive数据库，实现数据表创建，数据加载，以及数据查询。
实例代码如下：
<!-- more -->
```
package com.spark.sql

import org.apache.spark.sql.hive.HiveContext
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by Administrator on 2017/3/12.
  */
object HiveDataSource extends App{
  val conf = new SparkConf()
    .setAppName("HiveDataSource")
  val sc = new SparkContext(conf)
  val hiveContext = new HiveContext(sc)

  //创建student_infos表
  hiveContext.sql("DROP TABLE IF EXISTS STUDENT_INFO")
  //判断表是否存在，不存在则创建
  hiveContext.sql("CREATE TABLE IF NOT EXISTS STUDENT_INFO(NAME STRING,AGE INT)")
  //导入数据
  hiveContext.sql("LOAD DATA "
    + "LOCAL INPATH '/home/yangql/spark-study/sql/student_info.txt' "
    + "INTO TABLE STUDENT_INFO")

  //创建student_score表
  hiveContext.sql("DROP TABLE IF EXISTS STUDENT_SCORE")
  //判断表是否存在，不存在则创建
  hiveContext.sql("CREATE TABLE IF NOT EXISTS STUDENT_SCORE (NAME STRING,SCORE INT)")
  //导入数据
  hiveContext.sql("LOAD DATA "
    + "LOCAL INPATH '/home/yangql/spark-study/sql/student_score.txt' "
    + "INTO TABLE STUDENT_SCORE")
  //查询分数大于80的学生信息,并保存到good_student信息
  val goodStudentsDF=hiveContext.sql("SELECT T1.NAME,T1.AGE,T2.SCORE " +
    "FROM student_info T1 " +
    "INNER JOIN student_SCORE T2 " +
    "ON T1.NAME=T2.NAME " +
    "WHERE T2.SCORE>80")
  hiveContext.sql("DROP TABLE IF EXISTS GOOD_STUDENT")
  //goodStudentsDF.saveAsTable("GOOD_STUDENT")
  goodStudentsDF.write.saveAsTable("GOOD_STUDENT")

   //查询打印
  val goodStudentsRows=hiveContext.table("GOOD_STUDENT").collect()
  for(goodStudentsRow <- goodStudentsRows){
    println(goodStudentsRow)
  }
}

```
