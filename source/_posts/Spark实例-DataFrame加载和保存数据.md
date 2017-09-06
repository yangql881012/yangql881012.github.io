---
title: Spark实例-DataFrame加载和保存数据
date: 2017-05-07 09:54:59
tags: Spark
toc: true
categories: 大数据技术
---
- Spark加载不同格式文件时，调用sqlContext.read.format("").load方法
```
val peopleDF=sqlContext.read.format("json").load("E:\\spark\\src\\main\\resources\\people.json")
```
- Spark将DataFrame写入到文件中时，调用DF.write.format("").save方法
```
peopleDF.select("name")
   .write.format("parquet")
   //.mode(SaveMode.ErrorIfExists)
     .mode(SaveMode.Append)
   .save("E:\\spark\\src\\main\\resources\\people")
```
<!-- more -->
- 代码示例

```
package com.spark.sql

import org.apache.spark.sql.{SQLContext, SaveMode}
import org.apache.spark.{SparkConf, SparkContext}

/**
  * Created by Administrator on 2017/3/12.
  * 通用加载数据和保存数据
  * 文件保存模式
  * 1.SaveMode.ErrorIfExists
  * parquet文件的有点【列式存储】
  * 1.可以跳过不符合条件的数据，只读取需要的数据，降低IO量
  * 2.压缩编码可以降低磁盘的存储空间，由于同一类的数据类型是相同的，可以使用更高效的压缩编码，进一步节约存储空间
  * 3.只读取需要的列，支持向量运算，能够获得更好的扫描性能
  */
object GenericLoadAndSave extends App{
 val conf = new SparkConf()
    .setMaster("local")
    .setAppName("GenericLoadAndSave")
  val sc = new SparkContext(conf)
  val sqlContext= new SQLContext(sc)
  //加载数据
  val peopleDF=sqlContext.read.format("json").load("E:\\spark\\src\\main\\resources\\people.json")
  //保存到目录
  peopleDF.select("name")
    .write.format("parquet")
    //.mode(SaveMode.ErrorIfExists)
      .mode(SaveMode.Append)
    .save("E:\\spark\\src\\main\\resources\\people")
}

```
