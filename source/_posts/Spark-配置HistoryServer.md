---
title: Spark-配置HistoryServer
date: 2017-02-17 09:54:59
tags: HistoryServer
toc: true
categories: 大数据技术
---
## 1.为什么要有HistoryServer ##
以standalone运行模式为例，在运行Spark Application的时候，Spark会提供一个WEBUI列出应用程序的运行时信息；但该WEBUI随着Application的完成(成功/失败)而关闭，也就是说，Spark Application运行完(成功/失败)后，将无法查看Application的历史记录；

Spark history Server就是为了应对这种情况而产生的，通过配置可以在Application执行的过程中记录下了日志事件信息，那么在Application执行结束后，WEBUI就能重新渲染生成UI界面展现出该Application在执行过程中的运行时信息；

Spark运行在yarn或者mesos之上，通过spark的history server仍然可以重构出一个已经完成的Application的运行时参数信息（假如Application运行的事件日志信息已经记录下来）；
<!-- more -->
## 2.配置spark-defalut.conf ##
```
park.eventLog.enabled              true
spark.eventLog.dir                 hdfs://hadoop01:9000/input/sparklog
spark.yarn.historyServer.address   hadoop01:18080
spark.serializer                   org.apache.spark.serializer.KryoSerializer
spark.executor.instances           4
```
- spark.history.updateInterval  
默认值：10  
以秒为单位，更新日志相关信息的时间间隔  
- spark.history.retainedApplications  
默认值：50  
在内存中保存Application历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，当再次访问已被删除的应用信息时需要重新构建页面。  
- spark.history.ui.port  
默认值：18080  
HistoryServer的web端口  
- spark.history.kerberos.enabled  
默认值：false  
是否使用kerberos方式登录访问HistoryServer，对于持久层位于安全集群的HDFS上是有用的，如果设置为true，就要配置下面的两个属性  
- spark.history.kerberos.principal  
默认值：用于HistoryServer的kerberos主体名称  
- spark.history.kerberos.keytab  
用于HistoryServer的kerberos keytab文件位置  
- spark.history.ui.acls.enable  
默认值：false  
授权用户查看应用程序信息的时候是否检查acl。如果启用，只有应用程序所有者和spark.ui.view.acls指定的用户可以查看应用程序信息;否则，不做任何检查  
- spark.eventLog.enabled  
默认值：false  
是否记录Spark事件，用于应用程序在完成后重构webUI  
- spark.eventLog.dir  
默认值：file:///tmp/spark-events  
保存日志相关信息的路径，可以是hdfs://开头的HDFS路径，也可以是file://开头的本地路径，都需要提前创建  
- spark.eventLog.compress  
默认值：false  
是否压缩记录Spark事件，前提spark.eventLog.enabled为true，默认使用的是snappy  
<font color="red">注：</font>以spark.history开头的需要配置在spark-env.sh中的SPARK_HISTORY_OPTS，以spark.eventLog开头的配置在spark-defaults.conf  

## 3.配置spark-env.sh ##
```
export SPARK_HISTORY_OPTS="-Dspark.history.retainedApplications=10 -Dspark.history.fs.logDirectory=hdfs://hadoop01:9000/input/sparklog"
```
## 4.启动Spark History Server ##
```
start-history-server.sh
```
## 5.验证 ##
登陆http://192.168.1.231:18080/
## 6.HA下，Spark HistoryServer配置 ##
这里将HDFS的路径修改了，因为之前只有一个NN，HA的情况下，指定了两个，所以将hadoop01:9000替换成hadoop-cluster(hadoop/etc/hadoop/hdfs-site.xml 中dfs.nameservices配置的值),不需要指定端口

```
export SPARK_HISTORY_OPTS="-Dspark.history.retainedApplications=10 -Dspark.history.fs.logDirectory=hdfs://hadoop-cluster/spark/sparklogs"

```
