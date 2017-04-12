---
title: Spark-zip相关算子
date: 2017-03-29 15:54:59
tags: zip
toc: true
categories: Spark
---
## zip 算子 ##
zip函数用于将两个RDD组合成key/value形式的RDD，这里默认两个RDD的数量(partitions)以及元素数量都相同，否则将抛出异常` java.lang.IllegalArgumentException: Can't zip RDDs with unequal numbers of partitions:`
函数定义：
```
def zip[U: ClassTag](other: RDD[U]): RDD[(T, U)]
```
实例代码：
```
val conf = new SparkConf()
.setMaster("local")
.setAppName("ZipTest")
val sc = new SparkContext(conf)
val rdd1=sc.makeRDD(1 to 5,2)
val rdd2=sc.makeRDD(Seq("A","B","C","D","E"),2)
rdd1.zip(rdd2).foreach(print)
//output
//(1,A)(2,B)(3,C)(4,D)(5,E)
rdd2.zip(rdd1).foreach(print)
//output
//(A,1)(B,2)(C,3)(D,4)(E,5)
```
<!-- more -->
## zipPartitions 算子 ##
zipPartitions函数将多个RDD按照partition组合成为新的RDD，该函数需要组合的RDD具有相同的分区数，但对于每个分区内的元素数量没有要求。根据参数的个数可以分为三类，参数preservesPartitioning的作用：是否保留父RDD的partitioner分区信息。映射方法f参数N个RDD的迭代器。
- 参数是一个RDD：
```
def zipPartitions[B: ClassTag, V: ClassTag](rdd2: RDD[B], preservesPartitioning: Boolean)
def zipPartitions[B: ClassTag, V: ClassTag](rdd2: RDD[B])
```
RDD分区元素分布：
```
val rdd1=sc.makeRDD(1 to 5,2)
val rdd2=sc.makeRDD(Seq("A","B","C","D","E"),2)
val rdd1=sc.makeRDD(1 to 5,2)
//rdd1两个分区中元素分布：
   val rdd1ite=rdd1.mapPartitionsWithIndex((x,iter)=>{
     var result=List[String]()
     while(iter.hasNext){
       result ::= ("part_"+x+"|"+iter.next())
     }
     result.iterator
   }).collect()
//output:part_0|2,part_0|1,part_1|5,part_1|4,part_1|3
```
rdd1.zipPartitions元素分布:
```
rdd1.zipPartitions(rdd2){
  (rdd1Iter,rdd2Iter)=>{
     var result = List[String]()
     while(rdd1Iter.hasNext && rdd2Iter.hasNext){
        result::=(rdd1Iter.next() + "_" + rdd2Iter.next())
      }
    result.iterator
  }
}.collect().foreach(println)
//output
//2_B
//1_A
//5_E
//4_D
//3_C
```
- 参数是两个RDD：
```
def zipPartitions[B: ClassTag, C: ClassTag, V: ClassTag](rdd2: RDD[B], rdd3: RDD[C], preservesPartitioning: Boolean)  
def zipPartitions[B: ClassTag, C: ClassTag, V: ClassTag](rdd2: RDD[B], rdd3: RDD[C])
```
实例代码：
```
val rdd1=sc.makeRDD(1 to 5,2)
val rdd2=sc.makeRDD(Seq("A","B","C","D","E"),2)
val rdd3=sc.makeRDD(1 to 10,2)
rdd1.zipPartitions(rdd2,rdd3)(
    (iter1,iter2,iter3)=>{
      var result=List[String]()
      while(iter1.hasNext && iter2.hasNext && iter3.hasNext){
        result ::= iter1.next() + "_" + iter2.next() + "_" +iter3.next()
      }
      result.iterator
    }    
    ).collect().foreach(println)
//output
2_B_2
1_A_1
5_E_8
4_D_7
3_C_6
```
- 参数是三个RDD：
```
def zipPartitions[B: ClassTag, C: ClassTag, D: ClassTag, V: ClassTag](rdd2: RDD[B], rdd3: RDD[C], rdd4: RDD[D], preservesPartitioning: Boolean)
def zipPartitions[B: ClassTag, C: ClassTag, D: ClassTag, V: ClassTag](rdd2: RDD[B], rdd3: RDD[C], rdd4: RDD[D])
```
## zipWithIndex ##
该函数将RDD中的元素和这个元素在RDD中的ID（索引号）组合成键/值对。
```
def zipWithIndex(): RDD[(T, Long)]
```
代码示例:
```
val rdd1=sc.makeRDD(1 to 5,2)
rdd1.zipWithIndex().collect().foreach(println)
//output
(1,0)
(2,1)
(3,2)
(4,3)
(5,4)
```
## zipWithUniqueId ##
该函数将RDD中元素和一个唯一ID组合成键/值对，该唯一ID生成算法如下：
- 每个分区中第一个元素的唯一ID值为：该分区索引号，
- 每个分区中第N个元素的唯一ID值为：(前一个元素的唯一ID值) + (该RDD总的分区数)
```
def zipWithUniqueId(): RDD[(T, Long)]
```
代码示例：
```
val rdd1=sc.makeRDD(1 to 5,2)
rdd1.zipWithUniqueId().collect().foreach(println)
//output
(1,0)
(2,2)
(3,1)
(4,3)
(5,5)
```
rdd1总分区数为2,第一个分区第一个元素ID为0，第二个分区第一个元素ID为1,第一个分区第二个元素ID为0+2=2，第一个分区第三个元素ID为2+2=4,第二个分区第二个元素ID为1+2=3，第二个分区第三个元素ID为3+2=5.
