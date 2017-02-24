---
title: Spark-CentOS6下搭建spark-2.1.0-bin-hadoop2.7.md
date: 2017-02-13 10:54:59
tags: Spark2.1,hadoop2.7
categories: Spark
---
## 1.环境信息 ##

 - hadoop01:192.168.1.231
 - hadoop01:192.168.1.232
 - hadoop01:192.168.1.233
<!-- more -->
 ## 2.软件信息 ##
 - spark-2.1.0-bin-hadoop2.7
 - hadoop-2.7.2
 - scala-2.12.1
 - jdk1.8.0_91  

## 3.hadoop2.7.2安装 ##
参见文档《Hadoop2.5.1完全分布式集群部》  
 {% post_link Hadoop2.5.1完全分布式集群部署  Hadoop2.5.1完全分布式集群部署 %}

## 4.安装JDK1.8 ##
- 下载JDK包 jdk-8u91-linux-x64.tar.gz
- 将jdk-8u91-linux-x64.tar.gz包拷贝到 /opt目录并解压
```
mv /home/yangql/jdk-8u91-linux-x64.tar.gz /opt
tar -xvf jdk-8u91-linux-x64.tar.gz
```
- 将JDK环境配置到/etc/profile文件中
```
export JAVA_HOME=/opt/jdk1.8.0_91
export PATH=$PATH:$JAVA_HOME/bin
```
- source /etc/profile生效配置
- 检验配置是否正确
```
[root@hadoop01 etc]# java -version
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
[root@hadoop01 etc]#
```
## 5.安装Scala ##
- 下载安装包 scala-2.12.1.tgz
- 将scala-2.12.1.tgz包拷贝到/opt目录并解压
```
mv /home/yangql/scala-2.12.1.tgz /opt
tar -xvf scala-2.12.1.tgz
```
- 将Scala环境配置到/etc/profile文件中
```
export SCALA_HOME=/opt/scala-2.12.1
export PATH=$PATH:$SCALA_HOME/bin
```
- 环境验证
```
[root@hadoop01 etc]# scala -version
Scala code runner version 2.12.1 -- Copyright 2002-2016, LAMP/EPFL and Lightbend, Inc.
[root@hadoop01 etc]#
```
## 6.安装Spark ##
- 下载Spark安装包spark-2.1.0-bin-hadoop2.7.tgz
- 将spark-2.1.0-bin-hadoop2.7.tgz包拷贝到 /opt中并解压
```
mv /home/yangql/spark-2.1.0-bin-hadoop2.7.tgz /opt
tar -xvf spark-2.1.0-bin-hadoop2.7.tgz
```
- 修改配置文件spark-env.sh
```
cp spark-env.sh.template spark-env.sh
vim spark-env.sh
export SCALA_HOME=/opt/scala-2.12.1
export JAVA_HOME=/opt/jdk1.8.0_91
export SPARK_MASTER_IP=hadoop01
export SPARK_WORKER_MEMORY=512m
export HADOOP_CONF_DIR=/home/yangql/app/hadoop-2.7.2/etc/hadoop
```
- 修改配置文件slaves  
```
cp slaves.template slaves
vim slaves
hadoop02
hadoop03
```
## 7.scp到其它节点 ##
将在hadoop01上安装的jdk，scala，spark，hadoop等scp到节点hadoop02,hadoop03,如
```
scp /opt/jdk1.8.0_91 root@hadoop02:/opt
scp /opt/jdk1.8.0_91 root@hadoop03:/opt
scp /etc/profile root@hadoop02:/etc
scp /etc/profile root@hadoop03:/etc
scp /opt/scala-2.12.1 root@hadoop02:/opt
scp /opt/scala-2.12.1 root@hadoop03:/opt
scp /home/yangql/app/spark-2.1.0-bin-hadoop2.7 yangql@hadoop02:/home/yangql/app
scp /home/yangql/app/spark-2.1.0-bin-hadoop2.7 yangql@hadoop03:/home/yangql/app
```
## 8.启动并验证 ##
```
sh /home/yangql/app/hadoop-2.7.2/sbin/start-all.sh
sh /home/yangql/app/spark-2.1.0-bin-hadoop2.7/sbin/start-all.sh
jps
```
```
[yangql@hadoop01 ~]$ jps
2593 ResourceManager
2437 SecondaryNameNode
2679 Master
2247 NameNode
3215 Jps
```
```
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://hadoop01:7077 \
  --executor-memory 512m \
  /home/yangql/app/spark-2.1.0-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.1.0.jar \
  1000
```
