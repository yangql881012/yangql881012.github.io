---
title: Spark实例-操作KafKa数据
date: 2017-04-03 14:54:59
tags: Spark
toc: true
categories: 大数据技术
---
Spark操作kafka数据，有两种连接方式，直连Direct和Receiver方式
<!-- more -->
## 1.Direct 方式 ##
```
package com.spark.streaming

import org.apache.spark.SparkConf
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.Seconds
import org.apache.spark.streaming.kafka.KafkaUtils
import kafka.serializer.StringDecoder

/**
bin/kafka-topics.sh --create --zookeeper hadoop01:2181,hadoop02:2181,hadoop03:2181 --replication-factor 1 --partitions 1 --topic wordcount

bin/kafka-console-producer.sh --broker-list 192.168.1.231:9092,192.168.1.232:9092,192.168.1.233:9092 --topic wordcount
 */

object KafkaDirect extends App{
  val conf = new SparkConf()
  .setMaster("local[2]")
  .setAppName("KafkaDirect")
  val ssc = new StreamingContext(conf,Seconds(10))
  //创建一份kafka的参数
  val kafkaParams= Map[String,String]("metadata.broker.list"->"hadoop01:9092,hadoop02:9092,hadoop03:9092")
  //创建一个Set，里面放要读取的topic
  val topics = Set[String]("wordcount")

  val lineMap = KafkaUtils.createDirectStream[String,String,StringDecoder,StringDecoder](ssc, kafkaParams, topics)
  val lines = lineMap.map(_._2)

  val words = lines.flatMap(_.split(" "))

  val paris =words.map(word=>(word,1))
  val wordCounts = paris.reduceByKey(_+_)
  wordCounts.print
  ssc.start()
  ssc.awaitTermination()
  ssc.stop()

}
```
## 2.Receiver ##

```
package com.spark.streaming

import org.apache.spark.SparkConf
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.Seconds
import org.apache.spark.storage.StorageLevel
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.SparkContext
import org.apache.spark.streaming.Minutes


/**
 * 通过Kafka Receiver方法hi实现
 * 启动zooKeeper：sh zkServer.sh start
 * 启动Kafka：
 * bin/kafka-server-start.sh config/server.properties
 * 创建 topic
bin/kafka-topics.sh --create --zookeeper hadoop01:2181 --replication-factor 1 --partitions 1 --topic test
 查看：
bin/kafka-topics.sh --list --zookeeper hadoop01:2181
 发送消息
bin/kafka-console-producer.sh --broker-list 192.168.1.231:9092 --topic test
 接受消息
 bin/kafka-console-consumer.sh --bootstrap-server hadoop01:9092 --topic test --from-beginning
 */
object KafkaReciever extends App {
   val conf = new SparkConf()
  .setMaster("local[2]")
  .setAppName("KafkaDirect")
  val sc = new SparkContext(conf)
  val ssc = new StreamingContext(sc,Seconds(2))

  val zkQuorum = "hadoop01:2181,hadoop02:2181,hadoop03:2181"
  val group = "test-consumer-group"
  val topics = "test"
  val numThreads = 1

  val topicMap = topics.split(",").map((_,numThreads.toInt)).toMap
  val lineMap = KafkaUtils.createStream(ssc,zkQuorum,group,topicMap)
  val lines = lineMap.map(_._2)
  val words = lines.flatMap(_.split(" "))
  val pair = words.map(x => (x,1))
  val wordCounts = pair.reduceByKey(_+_)
  wordCounts.print
  ssc.checkpoint("E:/words/checkpoint")
  ssc.start
  ssc.awaitTermination
}
```
