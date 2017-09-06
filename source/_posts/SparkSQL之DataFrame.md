---
title: SparkSQL之DataFrame
date: 2017-03-15 08:54:59
tags: Spark
toc: true
categories: 大数据技术
---
## 1.DataFrame相关概念 ##
- Spark SQL是Spark中一个处理结构化数据的模块。它提供了被称为DataFrames的编程抽象，并能作为一个分布式的SQL查询引擎
- DataFrame是一个由命名列组成的分布式数据集。它从概念上讲相当于关系数据库里的一张表，或R/Python里的数据框架，但内部有很多优化。
- DataFrame能通过广泛的数据源构建，比如：结构化数据文件，Hive数据表，外部数据库或已有的RDDs。
- 使用SQLContext对象，应用程序能从已有RDD、从Hive数据表或从其它数据源创建DataFrame
<!-- more -->
## 2.DataFrame方法 ##
- 入口：val sc =  new SparkContext(conf)
- show：展示数据
```
df.show()
```
- show(numRows: Int) ：显示numRows条
```
df.show(2)
```
- show(truncate: Boolean) ：是否最多只显示20个字符，默认为true。
```
df.show(false)
```
- show(numRows: Int, truncate: Boolean) ：综合前面的显示记录条数，以及对过长字符串的显示格式。
```
df.show(2,false)
```
- collect：获取所有数据到数组，collect方法会将所有数据都获取到，并返回一个Array对象。
- collectAsList：获取所有数据到List
- describe(cols: String*)：获取指定字段的统计信息，这个方法可以动态的传入一个或多个String类型的字段名，结果仍然为DataFrame对象，用于统计数值类型字段的统计值，比如count, mean, stddev, min, max等。
```
df.describe("name","age").show(false)
+-------+---------------------------------+------------------+
|summary|name                             |age               |
+-------+---------------------------------+------------------+
|count  |3                                |2                 |
|mean   |null                             |24.5              |
|stddev |null                             |7.7781745930520225|
|min    |Andy                             |19                |
|max    |Michaeabcdefghijklmnopqrstuvwxyxz|30                |
+-------+---------------------------------+------------------+
```
- first:获取第一行记录
```
println(df.first())
[null,Michaeabcdefghijklmnopqrstuvwxyxz]
```
- head:获取第一行记录，head(n: Int)获取前n行记录
```
df.head(2).foreach(println)
[null,Michaeabcdefghijklmnopqrstuvwxyxz]
[30,Andy]
```
- take(n: Int):获取前n行数据
```
df.take(2).foreach(println)
[null,Michaeabcdefghijklmnopqrstuvwxyxz]
[30,Andy]
```
- takeAsList(n: Int):获取前n行数据，并以List的形式展现
```
println(df.takeAsList(2))
[[null,Michaeabcdefghijklmnopqrstuvwxyxz], [30,Andy]]
```
- where(conditionExpr: String)：SQL语言中where关键字后的条件传入筛选条件表达式，可以用and和or
```
df.where("name='Andy'").show(false)
+---+----+
|age|name|
+---+----+
|30 |Andy|
+---+----+
```
- filter：根据字段进行筛选
```
df.filter("name='Andy'").show(false)
+---+----+
|age|name|
+---+----+
|30 |Andy|
+---+----+
```
- select：获取指定字段值,根据传入的String类型字段名，获取指定字段的值，以DataFrame类型返回
```
df.select("name","age").show(false)
+---------------------------------+----+
|name                             |age |
+---------------------------------+----+
|Michaeabcdefghijklmnopqrstuvwxyxz|null|
|Andy                             |30  |
|Justin                           |19  |
+---------------------------------+----+
```
- selectExpr：可以对指定字段进行特殊处理 ,可以直接对指定字段调用UDF函数，或者指定别名等。传入String类型参数，得到DataFrame对象。
```
df.selectExpr("name","age as ageN","round(age)").show(false)
+---------------------------------+----+-------------+
|name                             |ageN|round(age, 0)|
+---------------------------------+----+-------------+
|Michaeabcdefghijklmnopqrstuvwxyxz|null|null         |
|Andy                             |30  |30           |
|Justin                           |19  |19           |
+---------------------------------+----+-------------+
```
- col：获取指定字段
```
println(df.col("name"))
name
```
- apply：获取指定字段 只能获取一个字段，返回对象为Column类型
```
 println(df.apply("name"))
 name
```
- drop：去除指定字段，保留其他字段 ,返回一个新的DataFrame对象，其中不包含去除的字段，一次只能去除一个字段。
```
df.drop("name").show(false)
+----+
|age |
+----+
|null|
|30  |
|19  |
+----+
```
- limit:limit方法获取指定DataFrame的前n行记录，得到一个新的DataFrame对象。和take与head不同的是，limit方法不是Action操作
。
```
 df.limit(2).show()
 +----+--------------------+
| age|                name|
+----+--------------------+
|null|Michaeabcdefghijk...|
|  30|                Andy|
+----+--------------------+
```
- orderBy和sort：按指定字段排序，默认为升序,加个-表示降序排序
```
df.orderBy(df.col("age")).show(false)
+----+---------------------------------+
|age |name                             |
+----+---------------------------------+
|null|Michaeabcdefghijklmnopqrstuvwxyxz|
|19  |Justin                           |
|30  |Andy                             |
+----+---------------------------------+
df.orderBy(- df.col("age")).show(false)
+----+---------------------------------+
|age |name                             |
+----+---------------------------------+
|null|Michaeabcdefghijklmnopqrstuvwxyxz|
|30  |Andy                             |
|19  |Justin                           |
+----+---------------------------------+
df.orderBy(df.col("age").desc).show(false)
+----+---------------------------------+
|age |name                             |
+----+---------------------------------+
|30  |Andy                             |
|19  |Justin                           |
|null|Michaeabcdefghijklmnopqrstuvwxyxz|
+----+---------------------------------+
```
- sortWithinPartitions :和上面的sort方法功能类似，区别在于sortWithinPartitions方法返回的是按Partition排好序的DataFrame对象。
- group by:根据字段进行group by操作 ,groupBy方法有两种调用方式，可以传入String类型的字段名，也可传入Column类型的对象。
```
df.groupBy("name")
```
- cube和rollup：group by的扩展
- GroupedData对象 :该方法得到的是GroupedData类型对象，在GroupedData的API中提供了group by之后的操作,max(colNames: String*)方法，获取分组中指定字段或者所有的数字类型字段的最大值，只能作用于数字型字段。min(colNames: String*)方法，获取分组中指定字段或者所有的数字类型字段的最小值，只能作用于数字型字段。mean(colNames: String*)方法，获取分组中指定字段或者所有的数字类型字段的平均值，只能作用于数字型字段。sum(colNames: String*)方法，获取分组中指定字段或者所有的数字类型字段的和值，只能作用于数字型字段。count()方法，获取分组中的元素个数

- distinct：返回一个不包含重复记录的DataFrame ,与dropDuplicates()方法不传入指定字段时的结果相同。
- dropDuplicates：根据指定字段去重 ，类似于select distinct a, b操作
```
df.distinct().show(false)
df.dropDuplicates().show(false)
```
- 聚合：聚合操作调用的是agg方法，该方法有多种调用方式。一般与groupBy方法配合使用。如：对id字段求最大值，对c4字段求和。
- unionALL：对两个DataFrame进行组合类似于SQL中的UNION ALL操作。
```
df.union(df1).show(false)
+----+---------------------------------+
|age |name                             |
+----+---------------------------------+
|null|Michaeabcdefghijklmnopqrstuvwxyxz|
|30  |Andy                             |
|19  |Justin                           |
|19  |Justin                           |
|null|Michaeabcdefghijklmnopqrstuvwxyxz|
|30  |Andy                             |
|19  |Justin                           |
|19  |Justin                           |
+----+---------------------------------+
```
- join(笛卡尔积):DF1.join(DF2)

- join(using一个字段形式 ):下面这种join类似于a join b using column1的形式，需要两个DataFrame中有相同的一个列名，
- join(using多个字段形式):除了上面这种using一个字段的情况外，还可以using多个字段，如下
```
df.join(df1).show(false)
df.join(df1,"name").show(false)
+---------------------------------+----+----+
|name                             |age |age |
+---------------------------------+----+----+
|Michaeabcdefghijklmnopqrstuvwxyxz|null|null|
|Andy                             |30  |30  |
|Justin                           |19  |19  |
|Justin                           |19  |19  |
|Justin                           |19  |19  |
|Justin                           |19  |19  |
+---------------------------------+----+----+
df.join(df1,Seq("name","age")).show(false)
+------+---+
|name  |age|
+------+---+
|Andy  |30 |
|Justin|19 |
|Justin|19 |
|Justin|19 |
|Justin|19 |
+------+---+
```
- 指定join类型 :两个DataFrame的join操作有inner, outer, left_outer, right_outer, leftsemi类型。在上面的using多个字段的join情况下，可以写第三个String类型参数，指定join的类型
```
df.join(df1,Seq("name","age"),"inner").show(false)
+------+---+
|name  |age|
+------+---+
|Andy  |30 |
|Justin|19 |
|Justin|19 |
|Justin|19 |
|Justin|19 |
+------+---+
```
- 使用Column类型来join:如果不用using模式，灵活指定join字段的话，可以使用如下形式
```
DF1.join(DF2 , DF1("id" ) === DF2( "t1_id"))
```
- 在指定join字段同时指定join类型:
```
DF1.join(DF2 , DF1("id" ) === DF2( "t1_id"), "inner")
```
- stat:获取指定字段统计信息,stat方法可以用于计算指定字段或指定字段之间的统计信息，比如方差，协方差等。这个方法返回一个DataFramesStatFunctions类型对象。
下面代码演示根据c4字段，统计该字段值出现频率在30%以上的内容。在jdbcDF中字段c1的内容为"a, b, a, c, d, b"。其中a和b出现的频率为2 / 6，大于0.3
```
DF.stat.freqItems(Seq ("c1") , 0.3).show()
```
- intersect:计算出两个DataFrame中相同的记录，
- except:获取一个DataFrame中有另一个DataFrame中没有的记录
- withColumnRenamed:重命名DataFrame中的指定字段名 ,如果指定的字段名不存在，不进行任何操作
```
DF.withColumnRenamed( "id" , "idx" )
```
- withColumn:whtiColumn(colName: String , col: Column)方法根据指定colName往DataFrame中新增一列，如果colName已存在，则会覆盖当前列。
- explode:行转列,有时候需要根据某个字段内容进行分割，然后生成多行，这时可以使用explode方法 ,下面代码中，根据c3字段中的空格将字段内容进行分割，分割的内容存储在新的字段c3_中，如下所示
```
DF.explode( "c3" , "c3_" ){time: String => time.split( " " )}
```

## 3.实例代码 ##

```
package com.spark.sql

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.sql.SQLContext
/**
  * 创建dataframe
  */
object DataFrameCreate extends App{
  val conf = new SparkConf()
    .setMaster("local")
    .setAppName("DataFrameCreate")
  //create a sc
  val sc =  new SparkContext(conf)
  //create sqlcontext
  val sqlContext=new SQLContext(sc)
  //create a dataframe
  val fileName="E:\\spark\\src\\main\\resources\\people.json"
  val df = sqlContext.read.json(fileName)
  val df1 = sqlContext.read.json(fileName)
  //print all the record ,including the head
  /*df.show()
  //print the schema info
  df.printSchema()
  //print name
  df.select("name", "age").show()
  //print age >20
  df.filter(df.col("age")>20).show
  //print name and age +1
  df.select(df.col("name"), df.col("age")+1).show()
  //group by age,and count
  df.groupBy("age").count().show()*/
  //df.show()
  //df.show(2,false)
  //df.show(false)
  //df.describe("name","age").show(false)
  //println(df.first())
  //df.head(2).foreach(println)
  //println(df.takeAsList(2))
  //df.where("name='Andy'").show(false)
  //df.filter("name='Andy'").show(false)
  //df.select("name","age").show(false)
  //df.selectExpr("name","age as ageN","round(age)").show(false)
  //println(df.col("name"))
  //println(df.apply("name"))
  //df.drop("name").show(false)
  //df.limit(2).show()
  //df.orderBy(df.col("age")).show(false)
  //df.orderBy(- df.col("age")).show(false)
  //df.orderBy(df.col("age").desc).show(false)
 // println(df.groupBy("name"))
  //df.distinct().show(false)
  //df.dropDuplicates().show(false)
  //df.union(df1).show(false)
  //df.join(df1).show(false)
  //df.join(df1,"name").show(false)
  df.join(df1,Seq("name","age"),"inner").show(false)
}
```
