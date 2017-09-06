---
title: Hive中的数据倾斜
date: 2017-06-15 23:54:59
tags: Hive
toc: true
categories: 大数据技术
---
## 1.空值数据倾斜 ##
场景：如日志中，常会有信息丢失的问题，比如全网日志中的user_id，如果取其中的user_id和users关联，会碰到数据倾斜的问题。  
解决方法1： user_id为空的不参与关联
```
Select * From log a
Join users b
On a.user_id is not null
And a.user_id = b.user_id
Union all
Select * from log a
where a.user_id is null;
```
解决方法2 ：赋与空值分新的key值  
```
Select *  
from log a
left outer join users b
on case when a.user_id is null thenconcat('hive',rand() ) else a.user_id end = b.user_id;
```
总结：  
方法2比方法效率更好，不但io少了，而且作业数也少了。方法1 log读取两次，jobs是2。方法2 job数是1 。这个优化适合无效id(比如-99,’’,null等)产生的倾斜问题。把空值的key变成一个字符串加上随机数，就能把倾斜的数据分到不同的reduce上 ,解决数据倾斜问题。

## 2.不同数据类型关联产生数据倾斜 ##
表与表关联时，如果关联字段之间数据类型不一致，可能会导致数据倾斜  
解决方法：转化数据类型，保持一致
## 3.Join的数据偏斜 ##
数据偏斜的原因包括以下两点：
- Map输出key数量极少，导致reduce端退化为单机作业。
- Map输出key分布不均，少量key对应大量value，导致reduce端单机瓶颈。
Hive中我们使用MapJoin解决数据偏斜的问题，即将其中的某个表（全量）分发到所有Map端进行Join，从而避免了reduce。这要求分发的表可以被全量载入内存。极限情况下，Join两边的表都是大表，就无法使用MapJoin。
解决方法：
- 情况1，考虑先对Join中的一个表去重，以此结果过滤无用信息。这样一般会将其中一个大表转化为小表，再使用MapJoin 。
- 情况2，考虑切分Join中的一个表为多片，以便将切片全部载入内存，然后采用多次MapJoin得到结果。
