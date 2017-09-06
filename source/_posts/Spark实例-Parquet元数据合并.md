---
title: Parquet元数据合并
date: 2017-05-07 13:54:59
tags: Spark
toc: true
categories: 大数据技术
---
当文件使用Parquet格式时，如果多次生成的文件列不同，可以进行元数据的合并，不用再像关系型数据库那样多个表关联。关键点`sqlContext.read.option("mergeSchema",true)`
<!-- more -->
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
