---
title: Spark-第一个Spark程序
date: 2017-02-15 10:54:59
tags: Spark
toc: true
categories: Spark
---
利用Scala语言编写第一个Spark程序，并分别在本地和Spark集群正常运行，主要有以下几个步骤。  
1 创建Spark的配置对象SparkConf，设置Spark的运行时信息，例如说通过色图Master来设置程序要链接到的集群的Master的URL，如果设置为local，则为本地运行。  
2 创建SparkContext对象，SparkContext是Spark程序所有功能的入口。其核心作用是：初始化Spark应用程序的运行所需要的核心组件。包括DAGSchedule，TaskSchedule，同时还负责Spark程序往Master注册等功能.  
3 根据数据的不同来源，通过SparkCont来创建RDD，数据被RDD划分为一系列的Partitions，分配到每一个Partition的数据属于一个Task处理的范围  
4 对初始的RDD进行Transformation级别的处理。例如map,filter等高阶函数的编程  
<!-- more -->
<font color="red">本地运行程序</font>  

```
package com.yangql.spark

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
object WordCount {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()//创建SparkConf对象
    conf.setAppName("My Fist Spark App")//设置应用程序的名称
    conf.setMaster("local")//程序在本地运行
    //创建SparkContext对象
    val sc = new SparkContext(conf)
    //读取本地的数据文件,并设置任务为1
    val lines=sc.textFile("D:/Program Files/spark-2.1.0-bin-hadoop2/spark-2.1.0-bin-hadoop2.7/README.md", 1)
    //处理数据
    val words=lines.flatMap(line=>line.split(" "))
    //println(words)
    val pairs=words.map(word=>(word,1))
    val wordCounts=pairs.reduceByKey(_+_)
    println(wordCounts)
    wordCounts.foreach(wordCountsPair=>println(wordCountsPair._1 +":"+ wordCountsPair._2))
    //关闭上下文
    sc.stop()
  }
}
```  
<font color="red">集群运行程序</font>  
```
package com.yangql.spark

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
object WordCount_Cluster {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()//创建SparkConf对象
    conf.setAppName("My Fist Spark App")//设置应用程序的名称
    //conf.setMaster("local")//程序在本地运行
    //创建SparkContext对象
    val sc = new SparkContext(conf)
    //读取本地的数据文件,并设置任务为1
    val lines=sc.textFile("/input/spark-test.txt", 1)
    //处理数据
    val words=lines.flatMap(line=>line.split(" "))
    //println(words)
    val pairs=words.map(word=>(word,1))
    val wordCounts=pairs.reduceByKey(_+_)
    println(wordCounts)
    wordCounts.collect.foreach(wordCountsPair=>println(wordCountsPair._1 +":"+ wordCountsPair._2))
    //关闭上下文
    sc.stop()
  }
}
```
