---
title: Spark琐碎记录
date: 2017-04-03 08:54:59
tags: 琐碎记录
toc: true
categories: Spark
---
本文主要针对Spark一些特性记录，随时保持更新
<!-- more -->
1. 写sc.textFile(file,num).cache()时，最好默认不设置num数，程序会根据系统环境自动去配置。还有cache缓存，如果RDD在后面只调用一次，可以不用添加缓存。（内存使用过多可能造成内存溢出）
2. 提交spark任务时，spark-submit --master yarn-cluster--driver-memory 5g --executor-memory 2g --num-executors 20 。除了设置资源大小，最好控制executors个数，防止程序无止境消耗内存资源量，影响其他调度正常运行
