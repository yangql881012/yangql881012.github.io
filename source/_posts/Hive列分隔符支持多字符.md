---
title: Hive列分隔符支持多字符
date: 2017-05-25 20:54:59
tags: Hive
toc: true
categories: 大数据技术
---
Hive 创建表默认的列分隔符是'\001',而且默认不支持多字符。查看网上大都说要重写InputFormat，但是没有成功。在stackoverflow上看到一个方法，亲测可以成功。在创建表时加上:
```
ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.MultiDelimitSerDe'
WITH SERDEPROPERTIES ("field.delim"="@|@")
STORED AS TEXTFILE
```
<!-- more -->

错误
```
Diagnostic Messages for this Task:
Error: java.lang.RuntimeException: Error in configuring object
	at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:112)
	at org.apache.hadoop.util.ReflectionUtils.setConf(ReflectionUtils.java:78)
	at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:136)
	at org.apache.hadoop.mapred.MapTask.runOldMapper(MapTask.java:449)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:343)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:164)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1657)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:158)
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:109)
	... 9 more
Caused by: java.lang.RuntimeException: Error in configuring object
	at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:112)
	at org.apache.hadoop.util.ReflectionUtils.setConf(ReflectionUtils.java:78)
	at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:136)
	at org.apache.hadoop.mapred.MapRunner.configure(MapRunner.java:38)
	... 14 more
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:109)
	... 17 more
Caused by: java.lang.RuntimeException: Map operator initialization failed
	at org.apache.hadoop.hive.ql.exec.mr.ExecMapper.configure(ExecMapper.java:137)
	... 22 more
Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: java.lang.ClassNotFoundException: Class org.apache.hadoop.hive.contrib.serde2.MultiDelimitSerDe not found
	at org.apache.hadoop.hive.ql.exec.MapOperator.getConvertedOI(MapOperator.java:329)
	at org.apache.hadoop.hive.ql.exec.MapOperator.setChildren(MapOperator.java:364)
	at org.apache.hadoop.hive.ql.exec.mr.ExecMapper.configure(ExecMapper.java:106)
	... 22 more
Caused by: java.lang.ClassNotFoundException: Class org.apache.hadoop.hive.contrib.serde2.MultiDelimitSerDe not found
	at org.apache.hadoop.conf.Configuration.getClassByName(Configuration.java:2101)
	at org.apache.hadoop.hive.ql.plan.PartitionDesc.getDeserializer(PartitionDesc.java:175)
	at org.apache.hadoop.hive.ql.exec.MapOperator.getConvertedOI(MapOperator.java:295)
	... 24 more

```

错误解决
```
hive add jar /home/yangql/app/hive-2.1.1/lib/hive-contrib-2.1.1.jar
```
