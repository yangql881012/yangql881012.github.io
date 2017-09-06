---
title: Spark实例-每天每个搜索词用户访问
date: 2017-05-07 08:54:59
tags: Spark
toc: true
categories: 大数据技术
---
## 1.需求分析 ##
根据平台的日志记录，统计处每天每个搜索词访问的UV，并把排名前三的搜索词及其UV打印出来。
## 2.数据格式 ##
日期  用户  搜索词 城市  平台  版本
<!-- more -->
```
2017-03-13,leo,barbecue,beijing,android,1.0
2017-03-13,leo,barbecue,beijing,android,1.0
2017-03-13,leo,barbecue,beijing,android,1.0
2017-03-13,leo,cloth,beijing,android,1.0
2017-03-13,leo2,cloth,beijing,android,1.0
2017-03-13,jack,barbecue,shanghai,android,1.1
2017-03-13,leo,paper,beijing,ios,1.0
2017-03-13,tom,barbecue,beijing,android,1.2
2017-03-13,leo,cup,beijing,android,1.0
2017-03-13,mary,barbecue,beijing,android,1.2
2017-03-13,leo,barbecue,beijing,ios,1.3
2017-03-13,leo,cup,beijing,android,1.0
2017-03-13,leo1,cup,beijing,android,1.0
2017-03-13,leo2,cup,beijing,android,1.0
2017-03-13,leo3,cup,beijing,android,1.0
2017-03-13,leo4,cup,beijing,android,1.0
```
## 3.实现分析 ##
- 针对原始数据（HDFS）获得输入的RDD
- 使用filter算子，去针对RDD输入的数据，进行数据过滤，过滤出符合条件的数据
- 将数据转换为“日期_搜索词，用户”格式，对他进行分组，对每天每个搜索词用户进行去重复操作，并统计去重后的数据，即为每天每个搜索词的UV
- 最后获得“日期_搜索词，UV”

## 4.代码实现 ##
```
package com.spark.sql

import org.apache.spark.sql.SQLContext
import org.apache.spark.sql.Row
import org.apache.spark.sql.types.{LongType, StringType, StructField, StructType}
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable.ListBuffer
object DailyTop3KeyWord extends App{
  val conf = new SparkConf()
    .setMaster("local")
    .setAppName("DailyTop3KeyWord")

  val sc = new SparkContext(conf)
  val sqlContext = new SQLContext(sc)
  //导入隐式转化
  import sqlContext.implicits._

  //伪造一份数据，查询条件，实际情况是通过MYSQL关系数据库查询
  var queryPara = Map(
    "city" -> List("beijing"),
    "platform" -> List("android"),
    "version" -> List("1.0","1.1","1.2","1.3")
  )

  //将查询条件封装为一个BroadCast变量
  val queryParaBroadCast = sc.broadcast(queryPara)

  //读取数据
  val searchLogRDD=sc.textFile("E:\\spark\\src\\main\\resources\\searchLog.txt")

  //使用广播变量进行筛选
  /**
  println(queryParaBroadCast.value.get("city"))
  val city = queryParaBroadCast.value.get("city").get
  val version = queryParaBroadCast.value.get("version").get
  println(version)
  if(version.contains("1.0")){
    println("1.0")
  }else{
    println("no")
  }
    */
    //通过广播变量筛选符合条件的数据
  val filtedSearchLogRDD=searchLogRDD.filter(log=>{
            val queryParamValue=queryParaBroadCast.value
            val citys=queryParamValue.get("city").get;
            val platforms=queryParamValue.get("platform").get;
            val versions=queryParamValue.get("version").get;
            val city=log.split(",")(3);
            val platform=log.split(",")(4);
            val version=log.split(",")(5);
            var flag=true;
            if(!citys.contains(city)){
              flag=false
            }
            if(!platforms.contains(platform)){
              flag=false
            }
            if(!versions.contains(version)){
              flag=false
            }
    flag
  })

  //将过滤出来的日志映射成“日期_搜索词,用户”
  val dateKeywordRDD=filtedSearchLogRDD.map(row=>(
           row.split(",")(0)+"_"+row.split(",")(2),row.split(",")(1)
    ))
  //dateKeywordRDD.foreach(println)
  //进行分组，获取每天每个个搜索词，有哪些用户搜索了，（没有去重）
  val dateKeywordUserRDD=dateKeywordRDD.groupByKey()
  dateKeywordUserRDD.collect().foreach(println)
  /**
(2017-03-13_cloth,CompactBuffer(leo, leo2))
(2017-03-13_cup,CompactBuffer(leo, leo, leo1, leo2, leo3, leo4))
(2017-03-13_barbecue,CompactBuffer(leo, leo, leo, tom, mary))
   */
 //将相同行数合并，并计算用户访问的次数，“日期_搜索词,UV”
  val dateKeywordUVRDD=dateKeywordUserRDD.map(row=>{
     val dateKeyWord=row._1
     val users=row._2.iterator
     val distinctUsers=new ListBuffer[String]
      while(users.hasNext){
       val user=users.next().toString()
        if(!distinctUsers.contains(user)){
          distinctUsers.append(user)
        }
    }
    val uv=distinctUsers.size
    (dateKeyWord,uv)
  })
  //将“日期_搜索词,UV”转换为DataFrame
  val dateKeywordUVRowRDD=dateKeywordUVRDD.map(row=>Row(row._1.split("_")(0),row._1.split("_")(1),row._2.toString.toLong))
  val structType=StructType(Array(
    StructField("date",StringType,true),
    StructField("keyword",StringType,true),
    StructField("uv",LongType,true)
  ))
  val dateKeywordUVDF=sqlContext.createDataFrame(dateKeywordUVRowRDD,structType)
  //注册临时函数
  dateKeywordUVDF.createOrReplaceTempView("daily_keyword_uv")
  //利用spark sql开窗函数，统计每天搜索UV排名前三的搜索词
   val dailyTop3KeyWordDF=sqlContext.sql(""
     + "select date," +
     "         keyword," +
     "          uv " +
     "  from  " +
     " (select " +
     "       date," +
     "       keyword," +
     "       uv," +
     "       row_number() over(partition by date order by uv desc ) rn " +
     " from daily_keyword_uv) " +
     "  where rn <=3")
  dailyTop3KeyWordDF.show()
  sc.stop()

}
```
