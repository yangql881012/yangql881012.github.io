---
title: Spark实例-通过HDFS文件实时统计
date: 2017-05-07 14:54:59
tags: Spark
toc: true
categories: 大数据技术
---
通过Spark Streaming，实时监控HDFS目录，发现有文件时，实时进行计算。
<!-- more -->
```
package com.spark.streaming

import org.apache.spark.SparkConf
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.Seconds

/*
 *
 * 通过HDFS文件实时统计
 */
object HDFSWordCount extends App {
  val conf = new SparkConf()
  .setAppName("HDFSWordCount")
  //.setMaster("hdfs://hadoop01:9000/")
  val ssc = new StreamingContext(conf,Seconds(5))

  val lines = ssc.textFileStream("hdfs://hadoop01:9000/wordcount_dir")
  val words = lines.flatMap(line=>line.split(" "))
  val paris = words.map((_,1))
  val wordCount=paris.reduceByKey(_+_)
  wordCount.print()
  ssc.start()
  ssc.awaitTermination()
  ssc.stop()
}
```
