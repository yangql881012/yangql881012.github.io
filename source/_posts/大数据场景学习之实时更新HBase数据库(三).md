---
title: 大数据场景学习之实时更新HBase数据库(三)
date: 2017-07-15 21:54:59
tags: 大数据场景学习
toc: true
categories: 大数据技术
---
## 一.业务场景 ##
1. 从Kafka中读取数据，并进行一定的加工
2. 利用phoenix将加载的数据更新到HBase中

## 二.开发环境 ##
1. MySQL
2. hive-2.1.1
3. hadoop-2.7.2
4. spark-2.1.0-bin-hadoop2.7
5. hbase-1.2.5
6. kafka-2.12
7. phoenix-4.10
<!-- more -->

## 三. Hive数据库相关表 ##
```
create table if not exists crm.bcst_t_ep_bal
(
rowkey           String  comment 'cust_no',
cust_no          String  comment '客户号                       ',
bal              Double  comment '存款余额                     '
)
comment '对公客户余额汇总信息'
--ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.MultiDelimitSerDe'
--WITH SERDEPROPERTIES ("field.delim"="@|@")
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,
cf:cust_no  ,
cf:bal      
")
TBLPROPERTIES ("hbase.table.name" = "crm:bcst_t_ep_bal", "hbase.mapred.output.outputtable" = "crm:bcst_t_ep_bal")
;
```

## 四.Kafka交互基础类 ##
```
package com.crm.query

import org.apache.spark.streaming.StreamingContext
import org.apache.spark.broadcast.Broadcast
import kafka.serializer.StringDecoder
import org.apache.spark.streaming.kafka.KafkaUtils

/**
 * 与kafka交互的基础类,
 */
trait SparkStreamingKafkaBase {

  /**
   * 初始化PhoenixBroadcast
   */
  def initPhoenixConnectorBroadcast(ssc:StreamingContext,phoenixUrl:String)={
    val phoenixConnectorBroadcast:Broadcast[PhoenixConnector]={
      println("phoenixUrl" + phoenixUrl)
      ssc.sparkContext.broadcast(PhoenixConnector(phoenixUrl))
    }
    phoenixConnectorBroadcast
  }

  /**
    * 初始化kafka读取
    */
  def initKafkaStream(ssc: StreamingContext,kafkaBroker:String, kafkaTopics:String) = {
    val kafkaParams= Map[String,String]("metadata.broker.list"->kafkaBroker)
    //创建一个Set，里面放要读取的topic
    val topics = kafkaTopics.split(",").toSet
    val kafkaStream = KafkaUtils.createDirectStream[String,String,StringDecoder,StringDecoder](ssc, kafkaParams, topics)
    kafkaStream
  }

}
```
## 五.Phoenix连接类 ##
```
package com.crm.query

import java.sql.DriverManager
import java.sql.Connection
import java.sql.Statement
import scala.collection.mutable.ArrayBuffer

class PhoenixConnector (createConnection: () => Connection) extends Serializable{

  lazy val conn = createConnection()

  def executeBatchUpdate(sqls:List[String]){
     var stat:Statement = null
     try{
       stat=conn.createStatement()
       sqls.foreach(sql=>{
         stat.addBatch(sql)
         println("sql=>" + sql)
       })
       stat.executeBatch()
       conn.commit()
     }catch{
       case e:Exception => println("PhoenixConnector.executeUpdate error :" + e.getMessage)
     }finally{
       if(stat !=null){
         stat.close()
       }
     }

   }
}

object PhoenixConnector{
   def apply(phoenixUrl: String): PhoenixConnector = {
    val createProducerFunc = () => {
      val conn = DriverManager.getConnection(phoenixUrl)
      conn.setAutoCommit(false)
      conn
    }
    new PhoenixConnector(createProducerFunc)
  }
    def main(args: Array[String]): Unit = {
      var arraybuffer = new ArrayBuffer[String]()

    }
}
```

## 六.实现类 ##
```
package com.crm.query

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.Seconds
import com.typesafe.config.ConfigFactory
import scala.collection.mutable.ArrayBuffer

/**
 * 通过spark steaming 读取kafka数据，并通过phoenix更新到Hbase中去
 */
object SparkStreamingHandlePhoenixCustBal extends SparkStreamingKafkaBase{
  //加载配置文件
  val config=ConfigFactory.load()
  def main(args: Array[String]): Unit = {
    //读取启动程序的参数
    val Array(
    batchDuration //时间窗口大小
    )
    =args


    // 获取配置信息
    val kafkaBroker=config.getString("kafka-broker")
    val kafkaTopics=config.getString("kafkaTopics")
    val phoenixUrl=config.getString("phoenix-url")
    val checkpointPath=config.getString("checkpoint-path")
    val hbaseTableName=config.getString("hbaseTableName")

    val conf= new SparkConf()
    .setMaster("local")
    .setAppName("SparkStreamingHandlePhoenixCustBal")
    val sc = new SparkContext(conf)
    //创建streamingcontext
    val ssc=new StreamingContext(sc,Seconds(batchDuration.toInt))
    //设置checkpoint
    //ssc.checkpoint(s"$checkpointPath/SparkStreamingHandlePhoenixCustBal")

     // 定义phoenix广播对象
    val phoenixBroadcast = initPhoenixConnectorBroadcast(ssc, phoenixUrl)
    // 读取kafka流数据
    val kafkaParams=""
    val topics=""
    val kafkaStream = initKafkaStream(ssc, kafkaBroker, kafkaTopics).map(_._2)
    //kafkaStream.print()
    /**
     * 消息格式 P00001@|@30000
     * val keyString=""
        val valueString=""
        val upsertSql = s"UPSERT INTO $hbaseTableName ($keyString) VALUES ($valueString)"
     */

    kafkaStream.foreachRDD(rdd=>{
     var batchSql= new ArrayBuffer[String]()
     rdd.collect().foreach(messages=>{
        println(messages)
        val custNo=messages.split("@\\|@")(0)
        val bal=messages.split("@\\|@")(1)
        //val keyString=Array("key","cust_no","bal").mkString(",")
        val keyString=config.getString("tblKeys")
        val valueString=Array("'"+custNo+"'","'"+custNo+"'","'"+bal+"'").mkString(",")
        println(custNo)
        println(bal)
        val upsertSql = s"UPSERT INTO $hbaseTableName($keyString) VALUES ($valueString)"

        println("upsertSql"+upsertSql)
        batchSql.append(upsertSql)
      })
      //println("batchSql " +batchSql.toList)
      val start=System.currentTimeMillis()
      if(batchSql.isEmpty){
        println("data is empty")
      }else{
        println(phoenixBroadcast.value)
        phoenixBroadcast.value.executeBatchUpdate(batchSql.toList)
      }

      var useTime = System.currentTimeMillis()-start
      println(s"batch update time $useTime")
    })

    ssc.start()
    ssc.awaitTermination()
    // 确保所有executor都执行完了才关闭程序
    sys.addShutdownHook({
      ssc.stop(true,true)
    })

  }
}
```
