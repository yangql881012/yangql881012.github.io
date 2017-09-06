---
title: Hbase-Spark读写HBase数据
date: 2017-07-16 23:54:59
tags: Hbase
toc: true
categories: 大数据技术
---
## 1.Spark写入数据到HBase ##
- HBase表结构
```
##创建hbase表
create 'user','basic'
```
- 定义HBase定义
```
val hbaseConf=HBaseConfiguration.create()
//zookeeper端口
hbaseConf.set("hbase.zookeeper.property.clientPort", "2181")
//zookeeper地址
hbaseConf.set("hbase.zookeeper.quorum", "hadoop01,hadoop02,hadoop03")
```
<!-- more -->
- 指定输出格式和输出表名
```
val jobConf=new JobConf(hbaseConf,this.getClass)
jobConf.setOutputFormat(classOf[TableOutputFormat])
jobConf.set(TableOutputFormat.OUTPUT_TABLE,"user")
```
- 定义转换格式  
```
将RDD[(uid:Int, name:String, age:Int)] 转换成 RDD[(ImmutableBytesWritable, Put)]
def convert(triple:(Int,String,Int))={
   val put=new Put(Bytes.toBytes(triple._1))
   put.addColumn(Bytes.toBytes("basic"), Bytes.toBytes("name"), Bytes.toBytes(triple._2))
   put.addColumn(Bytes.toBytes("basic"), Bytes.toBytes("age"), Bytes.toBytes(triple._3))
   (new ImmutableBytesWritable,put)
 }
```
- 保存
```
//使用saveAsHadoopDataset方法写入HBase
localData.saveAsHadoopDataset(jobConf)
```

## 2.Spark写HBase示例 ##  
```
package com.crm.base

import org.apache.spark.sql.SQLContext
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.sql.hive.HiveContext
import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.mapred.JobConf
import org.apache.hadoop.hbase.mapred.TableOutputFormat
import org.apache.hadoop.hbase.client.Put
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase.io.ImmutableBytesWritable

object TestHbaseWrite {
  def main(args: Array[String]): Unit = {
      val conf = new SparkConf()
      .setMaster("local")
      .setAppName("JDBCDataSrc")
    val sc = new SparkContext(conf)
    sc.setLogLevel("ERROR")
    val sqlContext=new SQLContext(sc)
    val hiveContext=new HiveContext(sc)

    //导入隐式转换
    import sqlContext.implicits._

    //定义HBase定义
    val hbaseConf=HBaseConfiguration.create()
    hbaseConf.set("hbase.zookeeper.property.clientPort", "2181")
    hbaseConf.set("hbase.zookeeper.quorum", "hadoop01,hadoop02,hadoop03")

    //指定输出格式和输出表名
    val jobConf=new JobConf(hbaseConf,this.getClass)
    jobConf.setOutputFormat(classOf[TableOutputFormat])
    jobConf.set(TableOutputFormat.OUTPUT_TABLE,"user")

    //read RDD data from somewhere and convert
    val rawData = List((1,"lilei",14), (2,"hanmei",18), (3,"someone",38))
    val localData= sc.parallelize(rawData).map(convert)

    //使用saveAsHadoopDataset方法写入HBase
    localData.saveAsHadoopDataset(jobConf)
  }


//将RDD[(uid:Int, name:String, age:Int)] 转换成 RDD[(ImmutableBytesWritable, Put)]
  def convert(triple:(Int,String,Int))={
    val put=new Put(Bytes.toBytes(triple._1))
    put.addColumn(Bytes.toBytes("basic"), Bytes.toBytes("name"), Bytes.toBytes(triple._2))
    put.addColumn(Bytes.toBytes("basic"), Bytes.toBytes("age"), Bytes.toBytes(triple._3))
    (new ImmutableBytesWritable,put)
  }
}
```

## 3.Spark读HBase数据 ##
Spark读取HBase，我们主要使用SparkContext 提供的newAPIHadoopRDDAPI将表的内容以 RDDs 的形式加载到 Spark 中。
- 定义HBase定义
```
val hbaseConf=HBaseConfiguration.create()
hbaseConf.set("hbase.zookeeper.property.clientPort", "2181")
hbaseConf.set("hbase.zookeeper.quorum", "hadoop01,hadoop02,hadoop03")
```
- 设置查询的表名
```
hbaseConf.set(TableInputFormat.INPUT_TABLE, "user")
val userRdd=sc.newAPIHadoopRDD(hbaseConf, classOf[TableInputFormat], classOf[ImmutableBytesWritable], classOf[Result])
```
- 遍历输出
```
userRdd.foreach{case (_,result)=>{
      val key=Bytes.toInt(result.getRow)
      val name=Bytes.toString(result.getValue("basic".getBytes, "name".getBytes))
      val age =Bytes.toInt(result.getValue("basic".getBytes, "age".getBytes))
      println(key)
      println(name)
      println(age)
    }}
```

## 4.park读HBase示例 ##

```
package com.crm.base
import org.apache.spark.sql.SQLContext
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.sql.hive.HiveContext
import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.hbase.mapred.TableInputFormatBase
import org.apache.hadoop.hbase.mapreduce.TableInputFormat
import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.client.Result
import org.apache.hadoop.hbase.util.Bytes

/**
 * Spark读取HBase，我们主要使用SparkContext 提供的newAPIHadoopRDDAPI将表的内容以 RDDs 的形式加载到 Spark 中。
 */
object TestHbaseRead {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
      .setMaster("local")
      .setAppName("JDBCDataSrc")
    val sc = new SparkContext(conf)
    sc.setLogLevel("ERROR")
    val sqlContext=new SQLContext(sc)
    val hiveContext=new HiveContext(sc)

    //导入隐式转换
    import sqlContext.implicits._

    //定义HBase定义
    val hbaseConf=HBaseConfiguration.create()
    hbaseConf.set("hbase.zookeeper.property.clientPort", "2181")
    hbaseConf.set("hbase.zookeeper.quorum", "hadoop01,hadoop02,hadoop03")

    //设置查询的表名
    hbaseConf.set(TableInputFormat.INPUT_TABLE, "user")
    val userRdd=sc.newAPIHadoopRDD(hbaseConf, classOf[TableInputFormat], classOf[ImmutableBytesWritable], classOf[Result])
    val count=userRdd.count()
    println(count)

    //遍历输出
    userRdd.foreach{case (_,result)=>{
      val key=Bytes.toInt(result.getRow)
      val name=Bytes.toString(result.getValue("basic".getBytes, "name".getBytes))
      val age =Bytes.toInt(result.getValue("basic".getBytes, "age".getBytes))
      println(key)
      println(name)
      println(age)
    }}

  }
}
```
