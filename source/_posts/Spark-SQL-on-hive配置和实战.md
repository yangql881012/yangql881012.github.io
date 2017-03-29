---
title: Spark SQL on hive配置和实战
date: 2017-03-04 08:54:59
tags: Spark
categories: Spark
---
## 1.安装配置Hive ##在Spark SQL下 ##
安装配置Hive并将元数据保存到数据库Mysql中，详见文档{% post_link Hive安装 Hive安装%}
<!-- more -->

## 2.在Spark SQL下配置hive-site.xml  ##
在master1上的spark-2.1.0-bin-hadoop2.7/conf目录创建hive-site.xml文件，内容如下：
```
<configuration>  
<property>
    <name>hive.metastore.uris</name>  
    <value>thrift://master1:9083</value>  
    <description>Thrift URI for the remote metastore. Used by metastore client to connect to remote metastore.</description>  
  </property>  
</configuration>  
```
## 3.配置驱动 ##
将$HIVE_HOME/lib/mysql-connector-java-5.1.40-bin.jar  中的mysql驱动拷贝到$SPARK_HOME/jars/下面即可。
## 4.启动hive的metastore后台进程  ##
hive --service metastore >> /home/yangql/app/hive-2.1.1/logs/metastore.log 2>& 1&
## 5.验证 ##
-  ./spark-shell --master spark://hadoop01:7077
- val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)
- hiveContext.sql("show databases").collect.foreach(println)
