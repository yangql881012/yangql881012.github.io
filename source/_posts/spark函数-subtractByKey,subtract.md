---
title: spark函数-subtractByKey,subtract
date: 2017-03-21 10:55:59
tags: wordCount
categories: Spark
---
## subtractByKey ##
RDD1.subtractByKey(RDD2):返回一个新的RDD，内容是：RDD1 key中存在的，RDD2 key中不存在的
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
  val rdd = sc.parallelize(Array((1,2),(3,4),(3,6),(9,10)))
  val other1 = sc.parallelize(Array((3,9)))
  rdd.subtractByKey(other1).foreach(println)
}
```
打印结果如下：
```
(1,2)
(9,10)
```

## subtract ##
RDD1.subtract(RDD2):返回一个新的RDD，内容是：RDD1中存在的，RDD2中不存在的  
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
  val rdd = sc.parallelize(Array((1,2),(3,4),(3,6),(9,10)))
  val other1 = sc.parallelize(Array((3,4)))
  rdd.subtract(other1).foreach(println)
}
```
打印结果如下：
```
(9,10)
(1,2)
(3,6)
```
