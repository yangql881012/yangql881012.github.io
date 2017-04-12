---
title: SparkSQL RDD转换DataFrame方法
date: 2017-03-31 08:54:59
tags: DataFrame
toc: true
categories: Spark
---
DataFrame可以创建临时视图，DataFrame相当于关系数据库上的一个表，我们可以利用DataFrame的这个特性，使用SQL语句进行一系列的操作。将SparkRDD转化为DataFram主要有两种方式。第一：通过反射的方式将RDD转换为DataFrame，第二：通过编程的方式将RDD转换为DataFrame
<!-- more -->

## 通过反射的方式将RDD转换为DataFrame ##
通过反射的方式将RDD转换为DataFrame，有以下几个步骤：
1. 定义一个case calss，成员包括需要使用的列，如：
```
case class Student(id:Int,name:String,age:Int)
```
2. 创建一个RDD中，数据是一个以case class 封装的对象,以case class reflection
```
val peoples=sc.textFile(fileName)
    .map(lines=>lines.split(","))
    .map(arrs=>Student(arrs(0).trim().toInt,arrs(1),arrs(2).trim().toInt))
```
3. 使用toDF函数，将RDD转换为DataFrame
```
val peopleDF=peoples.toDF()
```
4. 创建临时表
```
peopleDF.createOrReplaceTempView("people")
```
5. 使用sql对数据进行分析，返回一个DataFrame对象
```
val teenagerDF=sqlContext.sql("select * from people where age < 20")
```
6. 可以将DataFrame转换为RDD，使用RDD的特性
```
val teenagerRDD=teenagerDF.rdd
teenagerRDD.map(row=>Student(row(0).toString().toInt,row(1).toString(),row(2).toString().toInt))
    .collect()
    .foreach(stu=>println("id: "+stu.id + " name: " + stu.name + " age: "+stu.age))
```
7. 实例代码

```
package com.spark.sql

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.sql.SQLContext

/**
  * 通过反射的方式将RDD转换为DataFrame
  */
object RDD2DataFrameReflection extends App{
  val conf =  new SparkConf()
    .setMaster("local")
    .setAppName("RDD2DataFrameReflection")

  val sc = new SparkContext(conf)
  val  sqlContext= new SQLContext(sc)
  //导入隐式转化
  import sqlContext.implicits._
  //定义一个case class
  case class Student(id:Int,name:String,age:Int)
  //create a rdd,并且以case class reflection
  val fileName="E:/spark-scala-study/spark-scala-study/resouce/people.txt"
  val peoples=sc.textFile(fileName)
    .map(lines=>lines.split(","))
    .map(arrs=>Student(arrs(0).trim().toInt,arrs(1),arrs(2).trim().toInt))
  //convet to dataframe
  val peopleDF=peoples.toDF()
  //peopleDF.show()
  //create a tempory table
  peopleDF.createOrReplaceTempView("people")
  //create a new dataframe
  val teenagerDF=sqlContext.sql("select * from people where age < 20")
  //teenagerDF.show()
  //covert the dataframe to a rdd
  val teenagerRDD=teenagerDF.rdd
  //对RDD构造case class
  teenagerRDD.map(row=>Student(row(0).toString().toInt,row(1).toString(),row(2).toString().toInt))
    .collect()
    .foreach(stu=>println("id: "+stu.id + " name: " + stu.name + " age: "+stu.age))
  //row.getAs方法，对指定列名做操作
  teenagerRDD.map(row=>Student(row.getAs("id"),row.getAs("name"),row.getAs("age")))
    .collect()
    .foreach(stu=>println("id: "+stu.id + " name: " + stu.name + " age: "+stu.age))
  //getValuesmap
  teenagerRDD.map(row=>{
    val map=row.getValuesMap(Array("id","name","age"));
    Student(map("id"),map("name"),map("age"))
  })
    .collect()
    .foreach(stu=>println("id: "+stu.id + " name: " + stu.name + " age: "+stu.age))
}
```

## 通过编程的方式将RDD转换为DataFrame ##
1. 构造一个元素为Row的普通RDD,类：org.apache.spark.sql.Row
```
val peropleRDD = sc.textFile("E:\\spark\\src\\main\\resources\\people.txt")
   .map(line=>line.split(","))
   .map(arr=>Row(arr(0).toString.toInt,arr(1),arr(2).toString.toInt))
```
2. 编程方式动态构造元数据，使用类StructType，StructField
```
val peopleStruc=StructType(Array(
   StructField("id",IntegerType,true),
   StructField("name",StringType,true),
   StructField("age",IntegerType,true)
 ))
```
3. 将RDD转换位DataFrame
```
val peopleDF=sqlContext.createDataFrame(peropleRDD,peopleStruc)
```
4. 在DataFrame上使用SQL进行分析
```
val teenagerDF=sqlContext.sql("select * from people where age < 20")
```
5. 实例代码

```
package com.spark.sql
import org.apache.spark.sql.types.{IntegerType, StructField, StructType,StringType}
import org.apache.spark.sql.{Row, SQLContext}
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by Administrator on 2017/3/12.
  * 通过编程的方式将RDD转换为DataFrame
  */
object RDD2DataFrameProgrammatically  extends  App{
  val conf = new SparkConf()
    .setAppName("RDD2DataFrameProgrammatically")
    .setMaster("local")
  val sc = new SparkContext(conf)
  val sqlContext = new SQLContext(sc)

  //crate a RDD,构造元素为Row的普通RDD
  val peropleRDD = sc.textFile("E:\\spark\\src\\main\\resources\\people.txt")
    .map(line=>line.split(","))
    .map(arr=>Row(arr(0).toString.toInt,arr(1),arr(2).toString.toInt))
  //编程方式动态构造元数据
  val peopleStruc=StructType(Array(
    StructField("id",IntegerType,true),
    StructField("name",StringType,true),
    StructField("age",IntegerType,true)
  ))
  //将RDD转换位DataFrame
  val peopleDF=sqlContext.createDataFrame(peropleRDD,peopleStruc)
  peopleDF.show()
  //create a temporary table
  peopleDF.createOrReplaceTempView("people")
  //create a df
  val teenagerDF=sqlContext.sql("select * from people where age < 20")
  //create a rdd
  teenagerDF.rdd.collect().foreach(row => println(row))

}
```
