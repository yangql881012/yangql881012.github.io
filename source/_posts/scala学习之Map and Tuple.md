---
title: Scala学习之Map and Tuple
date: 2017-02-02 00:54:59
tags: Scala
toc: true
categories: 大数据技术
---
1.默认情况下Map构造的是不可变集合，里面的内容不能修改，一旦修改就变成新的Map，原有的Map保持不变。
2.Map的实例调用工厂方法apply来构造Map实例，
3.可变Map需要调用类：scala.collection.mutable.Map
4.如果想直接New出Map实例，需要使用HashMap等具体实现子类。
5.查询一个Map中的值用方法getOrElse，一方面如果值不存在会抛出异常。
6.scala.collection.immutable.SortedMap 可以得到排序的Map
7.交换key与value的值：val result = for((name,age) <- persInfo) yield(age,name)
8.scala.collection.mutable.LinkedHashMap 可以记住插入的顺序
9.Tuple中可以有很多不同类型的数据。Tuple可以作为函数的返回值
<!-- more -->
```
object HelloMapTuple {
  def main(args: Array[String]): Unit = {
    val bigData=Map("spark"->6,"hadoop"->5)

    val language = scala.collection.mutable.Map("Java"->10,"C"->0)
    language("Java")=11

    for((name,age)<-language){
      println(name)
      println(age)
    }
    //检索
    println(language.getOrElse("Python", ""))

    val persInfo = new scala.collection.mutable.HashMap[String,Int]
    //add and reduce
    persInfo += ("Java"->10,"C"->0)
    persInfo -= ("Java")
    for((name,age)<-persInfo){
      println(name + " " + age)
    }

    for(key <- persInfo.keySet) println(key)

    for(value <- persInfo.values) println(value)

    val result = for((name,age) <- persInfo) yield(age,name)
    for((name,age)<-result){
      println(name + " " + age)
    }

    val sortedMap=scala.collection.immutable.SortedMap(("Tom","10"),("Aim","6"),("Xary","8"))
    for((name,age)<-sortedMap){
      println(name + " " + age)
    }

    val linkedMap=scala.collection.mutable.LinkedHashMap(("Tom","10"),("Aim","6"),("Xary","8"))
    for((name,age)<-linkedMap){
      println(name + " " + age)
    }

    val bigTuple=("a","b","c","d","f")
    println(bigTuple._5)
    println(bigTuple._1)
  }
}
```
