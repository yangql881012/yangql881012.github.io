---
title: Spark SQL性能优化
date: 2017-03-17 08:54:59
tags: Spark
toc: true
categories: 大数据技术
---
## Spark SQL 性能优化的一些建议：##
1. 在` SQLContext.setConf()`中设置shuffle过程中的并行度，`spark.sql.shuffle.partitions`
2. 在Hive数据仓库建设过程中，合理设置数据类型，比如可设置为INT的，就不要设置为BIGINT。减少因为数据类型导致的不必要的开销
3. 编写SQL时，明确给出列名，不要写 `select *` 方式
4. 并行处理查询结果，对于Spark SQL的查询结果，如果数据量比较大，那么久不要一次性collect()到Driver再处理，使用foreach算子，并行处理查询结果。
5. 缓存表，对于一条SQL语句中可能多次使用到的表，可以对其进行缓存，使用 `SQLContext.cacheTable(TableName)`,或者`DataFrame.cache()`.Spark SQL 会用内存列存储的格式进行表的缓存。然后Spark SQL 就可以仅仅扫描需要使用的列，并且自动优化压缩，来最小化内存使用和GC开销。`SQLContext.uncacheTable(TableName)`可以将表从缓存中移除。用`SQLContext.setConf` 设置 `spark.sql.inMemoryColumnarStorage.batchSize`参数(默认10000)，可以配置列存储的单位。
6. 广播Join表，`spark.sql.autoBroadCastJoinThreshold`,默认(10M),在内存够用的情况下，提高其大小，可以将Join表的较小的表广播出去，而不用进行网络数据传输了。
7. 钨丝计划,`spark.sql.tungsten.enable`,默认是True，自动管理内存.
