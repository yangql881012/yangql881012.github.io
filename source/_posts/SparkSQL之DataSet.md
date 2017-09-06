---
title: SparkSQL之DataSet
date: 2017-04-01 14:54:59
tags: Spark
toc: true
categories: 大数据技术
---
## DataSet介绍 ##
DataSet是一个分布式数据集，从Spark 1.6开始引入，它集合了RDD API中的很多优点（强类型，lambda表达式），以及SparkSQL的优点（优化后的执行引擎）。DataSet可以通过JVM object来创建，然后通过transformation类的算子来进行操作。    
从Spark2.0后，SparkSQL的统一入口修改为SparkSession，SQLContext和HiveContext未来会被放弃掉。通过以下代码来创建SparkSession
<!-- more -->
```
val spark = SparkSession
      .builder()//用到了Java里面的构造器设计模式
      .appName("SparkSQLDemo")
      .master("local")
      //spark2.0必须设置spark.sql.warehouse.dir，需要设置spark.sql的元数据仓库的目录
      //默认已经启用
      .config("spark.sql.warehouse.dir", "E:\\worksplace\\spark\\spark-warehouse")
      //启用hive支持
      .enableHiveSupport()
      .getOrCreate()
      //导入隐式转换
  import spark.implicits._
```
## 代码示例 ##

```
package com.spark.dataset

import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.Row

/**
 * SparkSession,DataFrame,Dataset
 */
object SparkSQLDemo{
  //构造sparkSession
   case class Person(name:String,age:BigInt)
   case class Record(key:Int,value:String)

  def main(args: Array[String]): Unit = {
    val spark = SparkSession
       .builder()//用到了Java里面的构造器设计模式
       .appName("SparkSQLDemo")
       .master("local")
       //spark2.0必须设置spark.sql.warehouse.dir，需要设置spark.sql的元数据仓库的目录
       //默认已经启用
       .config("spark.sql.warehouse.dir", "E:\\worksplace\\spark\\spark-warehouse")
       //启用hive支持
       .enableHiveSupport()
       .getOrCreate()
   //导入隐式转换
     import spark.implicits._
     //读取json文件，构造一个untype弱类型的DataFrame
     val df = spark.read.json("E:\\worksplace\\spark\\src\\main\\resources\\people.json")
     df.show()//println打印数据
     df.printSchema()//打印元数据
     df.select("name").show()//select操作，典型的弱类型，untype操作
     df.select($"name", $"age"+1).show()
     df.filter($"age">19).show()//filter 过滤满足条件的数据
     df.groupBy("age").count().show()//group by 分组，再聚合
     //将df创建为临时视图，并使用sql查询
     df.createOrReplaceTempView("people")
     val sqlDf=spark.sql("select * from people")
     sqlDf.show()
     //基于jvm object来创建dataset
     //首先构造一个case class
         val caseClassDS=Seq(Person("yangql",18)).toDS()
     caseClassDS.show()
     //基于原始数据类型构造dataset
     val primitiveDS=Seq(0,1,2,3).toDS()
     primitiveDS.map(_+1).show()
     //基于已有的数据化文件，构造dataset
     //spark.read.json首先获得的是DataFrame，其次使用as[Person]之后，将一个DataFrame转换为dataset
     val peopleDS=spark.read.json("E:\\worksplace\\spark\\src\\main\\resources\\people.json").as[Person]
     peopleDS.show()
     //hive
     println("--=============hive=====================--")

     //创建Hive表
     spark.sql("create table if not exists src(key INT,value STRING)")
     spark.sql("LOAD DATA LOCAL INPATH E:\\worksplace\\spark\\src\\main\\resources\\people.json INTO TABLE SRC")
     spark.sql("select * from src").show()
     spark.sql("select count(*) from src").show()
     val hiveDF=spark.sql("select key,value from src where key<10 order by key")
     val hiveDS=sqlDf.map{
         case Row(key:Int,value:String)=>s"key:$key,value:$value"    
     }
     hiveDS.show()


     val recordDF=spark.createDataFrame((1 to 10).map(value=>{
       Record(value,s"val_$value")
     }))
     recordDF.show()
  }


}
```
