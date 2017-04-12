---
title: Spark-学习笔记-1
date: 2017-02-24 10:54:59
tags: Spark
toc: true
categories: Spark
---
Spark RDD 弹性，缓存，创建RDD的几种方式，Persist持久化，广播  

<!-- more -->
1.Spark RDD弹性：
- 自动的进行内存与磁盘数据的切换
- 基于LineAge的高效容错
- 任务失败会自动进行特定次数的重试
- Stage如果失败会自动进行特定次数的重试

2.什么时候使用缓存：
- 特别耗时
- 计算链很长
- checkpoint之后
- shuffle之后

3.创建RDD的几种方式：
- 使用程序中的集合创建RDD【主要用于测试】
- 使用本地文件系统创建RDD【测试大量数据文件】
- 使用HDFS创建RDD【生产环境最常用的RDD创建方式】
- 基于DB创建RDD
- 基于NOSQL创建RDD，如HBASE
- 基于S3创建RDD
- 基于数据流创建RDD

4.为什么要persist持久化：
- 某步骤计算特别耗时
- 计算链条特别长
- checkPoint之前
- shuffle之后(要进行网路传输)
- shuffle之前（框架默认把数据持久化到本地磁盘）

5.广播：  
默认情况下会考虑副本，更好的保持
广播时由Driver发给当前Application的所有Executor内存级别的全局变量。Executor中的线程池中的线程共享全局变量，极大地减少了网络传输（否则的话
每个Task要传输一次该变量。）并极大地节省了内存
