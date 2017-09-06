---
title: Spark实例-自定义聚合函数
date: 2017-05-07 14:54:59
tags: Spark
toc: true
categories: 大数据技术
---
Spark自定义聚合函数时，需要实现UserDefinedAggregateFunction中8个方法：
- inputSchema：输入的数据类型
- bufferSchema：中间聚合处理时，需要处理的数据类型
- dataType：函数的返回类型
- deterministic：是否是确定的
- initialize：为每个分组的数据初始化
- update：每个分组，有新的值进来时，如何进行分组的聚合计算
- merge：由于Spark是分布式的，所以一个分组的数据，可能会在不同的节点上进行局部聚合，就是update，但是最后一个分组，在各节点上的聚合值，要进行Merge，也就是合并
- evaluate：一个分组的聚合值，如何通过中间的聚合值，最后返回一个最终的聚合值
实例代码：
<!-- more -->

```
package com.spark.sql

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.sql.{Row, SQLContext}
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types._

/**
  * Created by Administrator on 2017/3/13.
  * 用户自定义聚合函数
  */
class StrCountUDAF extends  UserDefinedAggregateFunction{
  //输入的数据类型
  override def inputSchema: StructType = {
    StructType(Array(
      StructField("str",StringType,true)
    ))
  }
  //中间聚合处理时，所处理的数据类型
  override def bufferSchema: StructType = {
    StructType(Array(
      StructField("count",IntegerType,true)
    ))
  }
  //函数的返回类型
  override def dataType: DataType = {
    IntegerType
  }

  override def deterministic: Boolean = {
    true
  }
  //为每个分组的数据初始化
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer(0)=0
  }
  //指的是，每个分组，有新的值进来时，如何进行分组的聚合计算
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    buffer(0)=buffer.getAs[Int](0)+1
  }
  //由于Spark是分布式的，所以一个分组的数据，可能会在不同的节点上进行局部聚合，就是update
  //但是最后一个分组，在各节点上的聚合值，要进行Merge，也就是合并
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    buffer1(0)=buffer1.getAs[Int](0) + buffer2.getAs[Int](0)
  }
  //一个分组的聚合值，如何通过中间的聚合值，最后返回一个最终的聚合值
  override def evaluate(buffer: Row): Any = {
    buffer.getAs[Int](0)
  }
}

```
- 聚合函数的使用

```
package com.spark.sql

import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.sql.{Row, SQLContext}
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types._

object UDAF extends App{
  val conf = new SparkConf()
    .setMaster("local")
    .setAppName("DailyUVFunction")
  val sc = new SparkContext(conf)
  val sqlContext = new SQLContext(sc)
  //导入隐式转化
  import sqlContext.implicits._
  //构造用户的访问数据，并创建DataFrame
  val names=Array("tom","yangql","mary","test","test")
  val namesRDD = sc.parallelize(names)
  //将RDD转换为DataFram
  val namesRowRDD=namesRDD.map(name=>Row(name))
  val structType=StructType(Array(
    StructField("name",StringType,true)
  ))
  val namesDF=sqlContext.createDataFrame(namesRowRDD,structType)
  //注册表
  namesDF.createOrReplaceTempView("names")
  //定义和注册自定义函数
  sqlContext.udf.register("strCount",new StrCountUDAF)
  //使用自定义函数
  val df=sqlContext.sql("select name,strCount(name)  from names group by name")
  df.show()
}

```
