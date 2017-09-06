---
title: Spark-DataSet学习
date: 2017-05-09 08:54:59
tags: Spark
toc: true
categories: 大数据技术
---
## 1.DataSet相关概念 ##
Dataset是一个分布式的数据集。Dataset是Spark 1.6开始新引入的一个接口，它结合了RDD API的很多优点（包括强类型，支持lambda表达式等），以及Spark SQL的优点（优化后的执行引擎）。Dataset可以通过JVM对象来构造，然后通过transformation类算子（map，flatMap，filter等）来进行操作。Scala和Java的API中支持Dataset，但是Python不支持Dataset API。不过因为Python语言本身的天然动态特性，Dataset API的不少feature本身就已经具备了（比如可以通过row.columnName来直接获取某一行的某个字段）。R语言的情况跟Python也很类似。

Dataframe就是按列组织的Dataset。在逻辑概念上，可以大概认为Dataframe等同于关系型数据库中的表，或者是Python/R语言中的data frame，但是在底层做了大量的优化。Dataframe可以通过很多方式来构造：比如结构化的数据文件，Hive表，数据库，已有的RDD。Scala，Java，Python，R等语言都支持Dataframe。在Scala API中，Dataframe就是Dataset[Row]的类型别名。在Java中，需要使用Dataset<Row>来代表一个Dataframe。
<!-- more -->
## 2.DataSet操作 ##
- collect：将分布式存储在集群上的分布式数据集（比如dataset），中的所有数据都获取到driver端来
- first：获取数据集中的第一条数据
- persist()/cache():持久化，如果要对一个dataset重复计算两次的话，那么建议先对这个dataset进行持久化再进行操作，避免重复计算
- createTempView("employee")
- explain():答应执行计划，dataframe/dataset，比如执行了一个sql语句获取的dataframe，实际上内部包含一个logical plan，逻辑执行计划，设计执行的时候，首先会通过底层的catalyst optimizer，生成物理执行计划，比如说会做一些优化，比如push filter，还会通过whole-stage code generation技术去自动化生成代码，提升执行性能
- DataSet.write.save：将数据保存到指定目录
- printSchema()：打印结构
- 将DataFrame转化为DataSet
```
case class Employee(name: String, age: Long, depId: Long, gender: String, salary: Long)
val employeeDS=employee.as[Employee]
```
- coalesce和repartition:都是用来重新定义分区的,区别在于：coalesce，只能用于减少分区数量，而且可以选择不发生shuffle,repartiton，可以增加分区，也可以减少分区，必须会发生shuffle，相当于是进行了一次重分区操作
- distinct和dropDuplicates:都是用来进行去重的,distinct，是根据每一条数据，进行完整内容的比对和去重, dropDuplicates，可以根据指定的字段进行去重
```
val employeeDistinct=employeeDS.distinct()
 employeeDistinct.show()
 val employeeDropDup=employeeDS.dropDuplicates(Seq("name"))
 employeeDropDup.show()
```
- except：获取在当前dataset中有，但是在另外一个dataset中没有的元素
- filter：根据我们自己的逻辑，如果返回true，那么就保留该元素，否则就过滤掉该元素
- intersect：获取两个数据集的交集
```
employeeDS.except(employeeDS2).show()
employeeDS.intersect(employeeDS2).show()
employeeDS.filter(employee=>employee.age>35).show()
```
- map：将数据集中的每条数据都做一个映射，返回一条新数据
- flatMap：数据集中的每条数据都可以返回多条数据
- mapPartitions：一次性对一个partition中的数据进行处理
```
employeeDS.map(employee=>(
  employee.name,employee.salary,employee.salary+1000
)).show()
employeeDS.flatMap(employee=>Seq(
    (employee.name,employee.salary,employee.salary+1000),
    (employee.name,employee.salary,employee.salary+2000)
    )).show()
employeeDS.mapPartitions(employee=>{
  val result=scala.collection.mutable.ArrayBuffer[(String,Long,Long)]()
  while(employee.hasNext){
    var temp=employee.next()
    result += ((temp.name,temp.salary,temp.salary+5000))
  }
  result.iterator
}).show()
```
- joinWith,两个DataSet关联，指定关联条件
```
employee.joinWith(department, $"deptId" === $"id").show()
```
- sort：排序
```
employeeDS.sort($"salary".desc).show()
```
- randomSplit/sample
```
val employeeDSArr=employeeDS.randomSplit(Array(3,10,20))
employeeDSArr.foreach(ds=>ds.show())
employeeDS.sample(false, 0.3).show()
```
- groupBy/agg/avg/sum/max/min/count/countDistinct
```
employee
    .join(department, $"depId" === $"id")  
    .groupBy(department("name"))
    .agg(avg(employee("salary")), sum(employee("salary")), max(employee("salary")), min(employee("salary")), count(employee("name")), countDistinct(employee("name")))
    .show()
```
- collect_list/collect_set:collect_list就是将一个分组内，指定字段的值都收集到一起，不去重,collect_set，同上，但是唯一的区别是，会去重
```
/**
[1,WrappedArray(Leo, Jack),WrappedArray(Jack, Leo)]
[3,WrappedArray(Tom, Kattie),WrappedArray(Tom, Kattie)]
[2,WrappedArray(Marry, Jen, Jen),WrappedArray(Marry, Jen)]
*/
     employee.groupBy(employee("depId"))
     .agg(collect_list(employee("name")),collect_set(employee("name")))
     .collect()
     .foreach(println)
```
