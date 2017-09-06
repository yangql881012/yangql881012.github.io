---
title: Task未序列化(Task not serializable)问题分析及解决
date: 2017-04-03 09:54:59
tags: Spark
toc: true
categories: 大数据技术
---
在Spark程序中，在map等算子内部使用了外部定义的变量和函数，从而引发Task未序列化问题。其中最普遍的是当引用了某个类的成员函数或变量时，会导致这个类的所有成员（整个类）都需要支持序列化。虽然许多情形下，类使用了`extends Serializable`声明支持序列化，但是由于某些字段不支持序列化，仍然会导致整个类序列化时出现问题，最终导致出现Task未序列化问题。
出现如下错误如下：
<!-- more -->
```
Exception in thread "main" org.apache.spark.SparkException: Task not serializable
	at org.apache.spark.util.ClosureCleaner$.ensureSerializable(ClosureCleaner.scala:298)
	at org.apache.spark.util.ClosureCleaner$.org$apache$spark$util$ClosureCleaner$$clean(ClosureCleaner.scala:288)
	at org.apache.spark.util.ClosureCleaner$.clean(ClosureCleaner.scala:108)
	at org.apache.spark.SparkContext.clean(SparkContext.scala:2094)
	at org.apache.spark.rdd.RDD$$anonfun$flatMap$1.apply(RDD.scala:379)
	at org.apache.spark.rdd.RDD$$anonfun$flatMap$1.apply(RDD.scala:378)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:112)
	at org.apache.spark.rdd.RDD.withScope(RDD.scala:362)
	at org.apache.spark.rdd.RDD.flatMap(RDD.scala:378)
	at com.project.session.UserVisitAnalySpark$.randomExtractSession(UserVisitAnalySpark.scala:591)
	at com.project.session.UserVisitAnalySpark$.main(UserVisitAnalySpark.scala:105)
	at com.project.session.UserVisitAnalySpark.main(UserVisitAnalySpark.scala)
Caused by: java.io.NotSerializableException: com.project.dao.impl.SessionRandomExtractDAOImpl
Serialization stack:
	- object not serializable (class: com.project.dao.impl.SessionRandomExtractDAOImpl, value: com.project.dao.impl.SessionRandomExtractDAOImpl@76410abc)
	- field (class: com.project.session.UserVisitAnalySpark$$anonfun$27, name: sessionRandomExtractDAO$1, type: interface com.project.dao.SessionRandomExtractDAO)
	- object (class com.project.session.UserVisitAnalySpark$$anonfun$27, <function1>)
	at org.apache.spark.serializer.SerializationDebugger$.improveException(SerializationDebugger.scala:40)
	at org.apache.spark.serializer.JavaSerializationStream.writeObject(JavaSerializer.scala:46)
	at org.apache.spark.serializer.JavaSerializerInstance.serialize(JavaSerializer.scala:100)
	at org.apache.spark.util.ClosureCleaner$.ensureSerializable(ClosureCleaner.scala:295)
	... 12 more
```
此博客进行了原因分析以及解决办法：
<a>http://blog.csdn.net/sogerno1/article/details/45935159</a>
