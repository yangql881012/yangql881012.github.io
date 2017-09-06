---
title: spark实例-DataSet之统计部门员工平均薪资和平均年龄
date: 2017-05-08 10:54:59
tags: Spark
toc: true
categories: 大数据技术
---
## 需求分析 ##
计算部门的平均薪资和年龄  
- 只统计年龄在20岁以上的员工
- 根据部门名称和员工性别为粒度来进行统计
- 统计出每个部门分性别的平均薪资和年龄
<!-- more -->
## 关键技术点 ##
- 导入隐式转化`import spark.implicits._`
- 导入spark.sql.fucntions`import org.apache.spark.sql.functions._`
- 两个表的字段的连接条件，需要使用三个等号`$"depId" === $"id"`
- groupBy聚合时，指定表及相应字段`groupBy(department("name"), employee("gender"))`
- agg聚合函数`agg(avg(employee("salary")), avg(employee("age")))`
- dataframe == dataset[Row],dataframe的类型是Row，所以是untyped类型，弱类型,dataset的类型通常是我们自定义的case class，所以是typed类型，强类型
- dataset开发，与rdd开发有很多的共同点。dataset采用encoder序列化

## 代码示例 ##

```
package com.spark.dataset

import org.apache.spark.sql.SparkSession

/**
 * 计算部门的平均薪资和年龄
 *
 * 需求：
 * 		1、只统计年龄在20岁以上的员工
 * 		2、根据部门名称和员工性别为粒度来进行统计
 * 		3、统计出每个部门分性别的平均薪资和年龄
 *
 */
object DepartmentAvgSalaryAndAgeStat extends App{
  val spark=SparkSession
  .builder()
  .appName("DepartmentAvgSalaryAndAgeStat")
  .master("local")
  .config("spark.sql.warehouse.dir","E:\\worksplace\\spark\\spark-warehouse")
  .getOrCreate()
  //导入隐式转换
  import spark.implicits._
  //spark sql functions
  import org.apache.spark.sql.functions._
  /**
+---+--------------------+
| id|                name|
+---+--------------------+
|  1|Technical Department|
|  2|Financial Department|
|  3|       HR Department|
+---+--------------------+
   */
  val department=spark.read.json("E:\\worksplace\\spark\\src\\main\\resources\\department.json")
/**
+---+-----+------+------+------+
|age|depId|gender|  name|salary|
+---+-----+------+------+------+
| 25|    1|  male|   Leo| 20000|
| 30|    2|female| Marry| 25000|
| 35|    1|  male|  Jack| 15000|
| 42|    3|  male|   Tom| 18000|
| 21|    3|female|Kattie| 21000|
| 30|    2|female|   Jen| 28000|
| 19|    2|female|   Jen|  8000|
+---+-----+------+------+------+  
 */
  val employee=spark.read.json("E:\\worksplace\\spark\\src\\main\\resources\\employee.json")
  //department.show()
  //employee.show()
  //1.过滤20岁以上的员工
  val filtedEmployee=employee.filter("age>20")
  //filtedEmployee.show()
/**
 +---+-----+------+------+------+---+--------------------+
|age|depId|gender|  name|salary| id|                name|
+---+-----+------+------+------+---+--------------------+
| 25|    1|  male|   Leo| 20000|  1|Technical Department|
| 30|    2|female| Marry| 25000|  2|Financial Department|
| 35|    1|  male|  Jack| 15000|  1|Technical Department|
| 42|    3|  male|   Tom| 18000|  3|       HR Department|
| 21|    3|female|Kattie| 21000|  3|       HR Department|
| 30|    2|female|   Jen| 28000|  2|Financial Department|
+---+-----+------+------+------+---+--------------------+
 */
  // 注意：untyped join，两个表的字段的连接条件，需要使用三个等号
  val joined=filtedEmployee.join(department, $"depId" === $"id")
  val result=employee
  // 先对employee进行过滤，只统计20岁以上的员工
  .filter("age>20")
    // 需要跟department数据进行join，然后才能根据部门名称和员工性别进行聚合
    // 注意：untyped join，两个表的字段的连接条件，需要使用三个等号
  .join(department, $"depId" === $"id")
   // 根据部门名称和员工性别进行分组
  .groupBy(department("name"), employee("gender"))
   // 最后执行聚合函数
  .agg(avg(employee("salary")), avg(employee("age")))
  // 执行action操作，将结果显示出来
/**
+--------------------+------+-----------+--------+
|                name|gender|avg(salary)|avg(age)|
+--------------------+------+-----------+--------+
|       HR Department|female|    21000.0|    21.0|
|Technical Department|  male|    17500.0|    30.0|
|Financial Department|female|    26500.0|    30.0|
|       HR Department|  male|    18000.0|    42.0|
+--------------------+------+-----------+--------+
 */
  result.show()
}
```
