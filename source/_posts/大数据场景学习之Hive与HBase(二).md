---
title: 大数据场景学习之Hive与HBase(二)
date: 2017-07-15 20:54:59
tags: 大数据场景学习
toc: true
categories: 大数据技术
---
## 一.业务场景 ##
1. 从Hive中加载数据，并进行一定的加工
2. 将加载的数据更新到HBase中

## 二.开发环境 ##
1. MySQL
2. hive-2.1.1
3. hadoop-2.7.2
4. spark-2.1.0-bin-hadoop2.7
5. hbase-1.2.5
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

## 四.实现类 ##
```
package com.crm.query

import org.apache.spark.SparkConf
import org.apache.spark.sql.hive.HiveContext
import org.apache.spark.SparkContext
import com.typesafe.config.ConfigFactory
import org.apache.spark.sql.SQLContext
import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.mapred.JobConf
import org.apache.hadoop.hbase.mapred.TableOutputFormat
import org.apache.hadoop.hbase.client.Put
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.spark.sql.Row

object UpdateHbaseCustBal {
  def main(args: Array[String]): Unit = {
    val config = ConfigFactory.load("application.conf")

    val conf = new SparkConf()
      .setMaster("local")
      .setAppName("JDBCDataSrc")
    val sc = new SparkContext(conf)
    sc.setLogLevel("ERROR")
    val sqlContext=new SQLContext(sc)
    val hiveContext=new HiveContext(sc)

    //定义HBase定义
    val hbaseConf=HBaseConfiguration.create()
    hbaseConf.set("hbase.zookeeper.quorum", "hadoop01,hadoop02,hadoop03")
    hbaseConf.set("hbase.zookeeper.property.clientPort", "2181")

    //导入隐式转换
    import sqlContext.implicits._

    val sql="""select COALESCE(cust_no,'NULL') as rowkey,
                      COALESCE(cust_no,'NULL'),
                      sum(coalesce(bal,0)) as bal
                 from crm.bacc_t_ep_deps
                where trim(cust_no) is not null
             group by COALESCE(cust_no,'NULL')
            """
    //println(sql)
    val bacc_t_ep_depsDF=hiveContext.sql(sql)
    //bacc_t_ep_depsDF.show()

    //指定输出格式和输出表名
    val jobConf=new JobConf(hbaseConf,this.getClass)
    jobConf.setOutputFormat(classOf[TableOutputFormat])
    val outPutTbl=config.getString("updateHbaseCustBalOutPutTbl")
    jobConf.set(TableOutputFormat.OUTPUT_TABLE,outPutTbl)

    //更新数据

    val bacc_t_ep_depsRowRDD=bacc_t_ep_depsDF.rdd.map(row=>{
      (row(0).toString,row(1).toString,row(2).toString)
    }).map(convert).saveAsHadoopDataset(jobConf)

    sc.stop()
  }
  //rdd to hbase convert function
  def convert(tuple:(String,String,String))={
    println(tuple)
    val put=new Put(Bytes.toBytes(tuple._1))
    //列族，列名，具体值
    put.addColumn(Bytes.toBytes("cf"), Bytes.toBytes("cust_no"), Bytes.toBytes(tuple._2))
    put.addColumn(Bytes.toBytes("cf"), Bytes.toBytes("bal"), Bytes.toBytes(tuple._3))
    (new ImmutableBytesWritable,put)
  }
}
```
