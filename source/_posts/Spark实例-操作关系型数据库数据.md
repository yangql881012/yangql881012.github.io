---
title: Spark实例-操作关系型数据库数据
date: 2017-05-07 12:54:59
tags: Spark
toc: true
categories: 大数据技术
---
Spark操作关系型数据库数据，此处为MYSQL数据库。
<!-- more -->
```
package com.spark.sql
import org.apache.spark.sql.SQLContext
import org.apache.spark.sql.Row
import org.apache.spark.sql.types._
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by Administrator on 2017/3/12.
  */
object JDBCDataSrc extends  App{
  val conf = new SparkConf()
    .setMaster("local")
    .setAppName("JDBCDataSrc")
  val sc = new SparkContext(conf)
  val sqlContext=new SQLContext(sc)
  import sqlContext.implicits._
  val url="jdbc:mysql://localhost:3306/sparkpro"
  val userName="root"
  val password="root"
  //创建Dataframe
  val studenInfoDF=sqlContext.read.format("jdbc").options(Map(
    "url"->url,
    "dbtable"->"student_info",
    "user"->userName,
    "password"->password
  )).load()
  studenInfoDF.show()

  //创建Dataframe
  val studentScoreDF=sqlContext.read.format("jdbc").options(Map(
    "url"->url,
    "dbtable"->"student_score",
    "user"->userName,
    "password"->password
  )).load()
  //studentScoreDF 转换为RDD，并过滤出分数大于80分的学生
  val goodStudentRDD=studentScoreDF.rdd.filter(row=>(row.getAs[Int]("score")>=80))
 // for (elem <- goodStudentRDD.collect()) {
 //   print(elem)
 // }

  //a RDD for studenfInfo
  val studenInfoRDD=studenInfoDF.rdd.map(row=>(row.getAs[String]("name"),row.getAs[Int]("age")))
    .join(goodStudentRDD.map(row=>(row.getAs[String]("name"),row.getAs[Int]("score"))))
  val studenInfoRowRDD=studenInfoRDD.map(row=>Row(row._1,row._2._1.toString.toLong,row._2._2.toString.toLong))
  //studenInfoRDD.foreach(println)
  //将RDD转化为DataFrame
   val studentStruct=StructType(Array(
    StructField("name",StringType,true),
    StructField("age",LongType,true),
    StructField("score",LongType,true)
  ))
  val studentStructDF=sqlContext.createDataFrame(studenInfoRowRDD,studentStruct)
  studentStructDF.write.saveAsTable("good_student")
  //将数据插入到数据库中
}

```
