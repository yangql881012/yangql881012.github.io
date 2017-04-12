---
title: SparkSQL之根据不同数据来源创建DataFrame
date: 2017-04-01 08:54:59
tags: DataFrame
toc: true
categories: Spark
---
## 1. 介绍 ##
本文主要介绍根据不同数据来源创建DataFrame，主要有JDBC数据源，Hive数据源，JSON数据源，Parquet数据源。
<!-- more -->
## 2. JDBC数据源创建DataFrame ##
关键点：
- 连接数据库
- 使用SQLContext.read.format("jdbc")
- options 参数配置
```
options(Map(
    "url"->url,
    "dbtable"->"student_score",
    "user"->userName,
    "password"->password
  ))
```
- 将DataFrame数据保存到关系数据库中
```
studentStructDF.write.saveAsTable("good_student")
```
- 实例代码:

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
## 3. Hive 数据源创建DataFrame ##
关键点：
- 创建HiveContext
- hiveContext.sql() 方法创建表，删除表，加载数据，查询SQL返回DataFrame
- 将数据保存到Hive中
```
 goodStudentsDF.write.saveAsTable("GOOD_STUDENT")
```
- 示例代码:

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
## 4. JSON数据源创建DataFrame ##
关键点：
- sqlContext.read.json()方法读取Json文件，创建DataFrame
- 示例代码

```
package com.spark.sql

import org.apache.spark.sql.{Row, SQLContext}
import org.apache.spark.sql.types._
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by Administrator on 2017/3/12.
  * 从JSON数据源中读取数据
  */
object JsonDataSource extends  App{
  val conf = new SparkConf()
    .setMaster("local")
    .setAppName("JsonDataSource")
  val sc = new SparkContext(conf)
  val sqlContext = new SQLContext(sc)
  //导入隐式转化
  import  sqlContext.implicits._
  //创建学生成绩DataFrame
  val studentScoreDF=sqlContext.read.json("E:\\spark\\src\\main\\resources\\studentscore.json")
  //create a temporary table
  studentScoreDF.createOrReplaceTempView("student_score")
  val goodStudentScoreDF = sqlContext.sql("select * from student_score where score >= 80")
  val goodStudentName = goodStudentScoreDF.rdd.map(row => row(0)).collect()
  goodStudentName.foreach(println)
  //googStudentScoreDF.show()
  //创建学生信息DataFrame
  val studentInfoJson=Array("{\"name\":\"Tom\",\"age\":10}" +
    "{\"name\":\"Tom1\",\"age\":11}" +
    "{\"name\":\"Tom2\",\"age\":12}" +
    "{\"name\":\"Tom3\",\"age\":13}")
  val studentInfoDF=sqlContext.read.json(sc.parallelize(studentInfoJson))
  //create a temporary table of student info
  studentInfoDF.createOrReplaceTempView("student_info")
  var sql = "select name,age from student_info where name in ("
  for(i <- 0 until goodStudentName.length){
    sql += "'" + goodStudentName(i) + "'"
    if(i < goodStudentName.length-1){
     sql += ","
    }
  }
  sql += ")"
  println("sql========" + sql)
  val goodStudentInfoDF=sqlContext.sql(sql)
  //将分数大于80分的学生成绩信息与基本信息进行Join
  val goodStudentRDD = goodStudentScoreDF.rdd.map(row=>(row.getAs[String]("name"),row.getAs[Long]("score")))
    .join(goodStudentInfoDF.rdd.map(row => (row.getAs[String]("name"),row.getAs[Long]("age"))))
  //将RDD转换为DataFrame
  val goodStudentRowRDD=goodStudentRDD.map(row=>Row(row._1,row._2._1.toString().toLong,row._2._2.toString.toLong))
  val studentStruct=StructType(Array(
    StructField("name",StringType,true),
    StructField("score",LongType,true),
    StructField("age",LongType,true)
  ))

  val goodStudentDF=sqlContext.createDataFrame(goodStudentRowRDD,studentStruct)
  goodStudentDF.show()
}
```
## 5. Parquet数据源创建DataFrame ##
关键点：
- sqlContext.read.parquet()方法创建DataFrame
- 将DataFrame数据写入到Parquet
```
studentWithNameAndAgeDF.write.format("parquet").mode(SaveMode.Append)
    .save("E:\\spark\\src\\main\\resources\\student")
```
- 示例代码

```
package com.spark.sql

import org.apache.spark.sql.SQLContext
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by Administrator on 2017/3/12.
  */
object ParquetLoadData extends  App{
  val conf = new SparkConf()
    .setMaster("local")
    .setAppName("ParquetLoadData")
  val sc = new SparkContext(conf)
  val sqlContext=new SQLContext(sc)

  val peopleDF=sqlContext.read.parquet("E:\\spark\\src\\main\\resources\\people")
  peopleDF.show()
  //将dataframe转化为RDD并打印出来
  peopleDF.rdd.map(row => "name:" + row(0))
      .collect()
      .foreach(username => println(username))
}

```
- 用mergeSchema的方式读取数据，进行元数据的合并
```
val studentsDF=sqlContext.read.option("mergeSchema",true).parquet("E:\\spark\\src\\main\\resources\\student")
```
- 实例代码

```
package com.spark.sql

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.sql.{SQLContext, SaveMode}

/**
  * Created by Administrator on 2017/3/12.
  * parquet数据元合并元数据
  */
object ParquetSchemaMerge extends App{
  val conf = new SparkConf()
    .setMaster("local")
    .setAppName("ParquetLoadData")
  val sc = new SparkContext(conf)
  val sqlContext=new SQLContext(sc)
  //导入隐式转换
  import sqlContext.implicits._
  //创建第一个RDD
  val studentWithNameAndAge=Array(("Tom",13),("Mary",14))
  val studentWithNameAndAgeDF=sc.parallelize(studentWithNameAndAge,1).toDF("name","age")
  //保存
  studentWithNameAndAgeDF.write.format("parquet").mode(SaveMode.Append)
    .save("E:\\spark\\src\\main\\resources\\student")

  //创建第二个RDD
  val studentWithNameAndGrade=Array(("Yangql","A"),("Test","B"))
  val studentWithNameAndGradeDF=sc.parallelize(studentWithNameAndGrade).toDF("name","grade")
  studentWithNameAndGradeDF.write.format("parquet").mode(SaveMode.Append)
    .save("E:\\spark\\src\\main\\resources\\student")

  //用mergeSchema的方式读取数据，进行元数据的合并
  val studentsDF=sqlContext.read.option("mergeSchema",true).parquet("E:\\spark\\src\\main\\resources\\student")
  studentsDF.show()
  studentsDF.printSchema()
}
```
