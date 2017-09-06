---
title: spark函数-cogroup
date: 2017-03-21 10:54:59
tags: Spark
toc: true
categories: 大数据技术
---
cogroup：将多个RDD中同一个Key对应的Value组合到一起。最多可以组合<font color='red'>四个</font>RDD。当对应RDD中不存在对应Key的元素（自然就不存在Value了），在组合的过程中将RDD对应的位置设置为CompactBuffer()了，而不是去掉了。
实例如下：
<!-- more -->
```
package com.spark.core

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext

object RDDTest extends App{
  val conf = new SparkConf()
  .setMaster("local")
  .setAppName("RDDTest")
  val sc  = new SparkContext(conf)
  val rdd = sc.parallelize(Array((1,2),(3,4),(3,6)))
  val other1 = sc.parallelize(Array((3,9)))
  val other2 = sc.parallelize(Array((4,9)))
  val other3 = sc.parallelize(Array((6,9)))
  rdd.cogroup(other1,other2,other3).foreach(println(_))
}
```
打印结果如下：
```
(4,(CompactBuffer(),CompactBuffer(),CompactBuffer(9),CompactBuffer()))
(1,(CompactBuffer(2),CompactBuffer(),CompactBuffer(),CompactBuffer()))
(6,(CompactBuffer(),CompactBuffer(),CompactBuffer(),CompactBuffer(9)))
(3,(CompactBuffer(4, 6),CompactBuffer(9),CompactBuffer(),CompactBuffer()))
```
