---
title: Spark二次排序
date: 2017-04-03 08:54:59
tags: Spark
toc: true
categories: 大数据技术
---
Spark `sortByKey`只针对单个字段，如果要对多个字段进行排序，就只能自己单独实现一个排序类。关键点：
- 继承`Ordered[T]`接口
- 继承`Serializable`
- 实现compare方法
- 使用时需要实例化定义的排序类  
实现三个字段A,B,C排序，先按照A排序，如果A相同，再以B排序，B相同，以C排序。代码如下：
<!-- more -->

```
package com.spark.core

import org.apache.spark.sql.SparkSession

/**
 * 二次排序
 */
class SecondSortKey(val first:Int,val second:Int,val third:Int) extends Ordered[SecondSortKey] with Serializable{
  def compare(that:SecondSortKey):Int={
     if(this.first-that.first!=0){
       this.first-that.first
     }else if(this.second-that.second!=0){
       this.second - that.second
     }else{
       this.third-that.third
     }
 }
}
object SecondSortKey{
  def main(args: Array[String]): Unit = {
    val spark=SparkSession
    .builder().appName("SecondSortKey")
    .master("local")
    .getOrCreate()
    import spark.implicits._
    //读取文件，生成RDD
    val lines=spark.sparkContext.textFile("E:\\worksplace\\spark\\src\\main\\resources\\sort.txt")
    lines.foreach(println)
    /**
    1 1 3
    1 1 2
    3 6 1
    1 3 4
    2 1 6
     */
    //<SecondSortKey,line>
    val paris=lines.map(line=>(
      new SecondSortKey(line.split(" ")(0).toInt,
                        line.split(" ")(1).toInt,
                        line.split(" ")(2).toInt),
      line
    ))
    //排序
    val sortedParis=paris.sortByKey()
    //将排序后的行映射回来
    val sortedLines=sortedParis.map(paris=>(paris._2))
    //打印
    sortedLines.foreach(println)
    //output
    /**
     *  1 1 2
        1 1 3
        1 3 4
        2 1 6
        3 6 1
     */
  }
}
```
