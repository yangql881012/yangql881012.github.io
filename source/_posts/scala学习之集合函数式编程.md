---
title: scala学习之集合函数式编程
date: 2017-02-10 13:54:59
tags: 函数式编程
categories: scala
---
1.在scala中，Iterable是公共的方法   	，Iterable要求继承者实现一些公共的方法。
2.Array是一个非常基础的数据结构，不属于scala集合体系。  
3.集合分为可变与不可变之分，不可变的集合在scala.collection.immutable保中，可变集合在scala.collection.mutable中  
4.List是元素的列表集合，是不可变的。  
5.List中head是第一个元素，tail是指剩下元素的集合的集合  
6.操作符"::"把List和其它的元素进行组拼来构建新的List  
7.如果集合中没有元素，返回Nil  
8.LinkedList是元素可变的列表  
9.Set是元素不可重复的，且元素是无序的。HashSet元素不可变，也不保证顺序。SortedSet会排序  
10.map与flatMap的区别：Spark 中 map函数会对每一条输入进行指定的操作，然后为每一条输入返回一个对象；  
而flatMap函数则是两个操作的集合——正是“先映射后扁平化”：
- 操作1：同map函数一样：对每一条输入进行指定的操作，然后为每一条输入返回一个对象
- 操作2：最后将所有对象合并为一个对象
11.只有一个原始时，可以用"\_"占位符替代，简化函数编程。如：.reduce(_ +\_); //reduce((x,y) => x + y)  
<!-- more -->
```
object HelloFunctional_Iterable {
  def main(args: Array[String]): Unit = {
      val names = List(1,2,3,4,5)
      //println(names.head)
      //println(names.tail)
      //println(5::names)
      var linkedList=scala.collection.mutable.LinkedList(2,3,4,5)
      //println(linkedList.head)
      while(linkedList != Nil) {
        //println(linkedList.elem)
        linkedList=linkedList.tail
      }

      val set=Set(1,2,3,5)
      //println(set)
      set + 1
      set + 7
      //println(set)

      val hashSet = scala.collection.mutable.HashSet(1,2,3)
      val linkedHashSet = scala.collection.mutable.LinkedHashSet(1,2,3)
      val sortedSet=scala.collection.mutable.SortedSet(1,2,4,3)
      //println(sortedSet)
      println("=====================================================")
      val list=List("I am so pround of spark",
                     "spark is very interested")
                                               .flatMap(x => x.split(" "))
                                               .map(x => (x,1))
                                               .map(x => (x._2))
                                               .reduce(_ + _); //reduce((x,y) => x + y)
      val list5=List("I am so pround of spark",
                     "spark is very interested")
                                               .flatMap(_.split(" "))
                                               .map((_,1))
                                               .map((_._2))
                                               .reduce(_ + _); //reduce((x,y) => x + y)
      println(list)
      val list1=List("I am so pround of spark",
                     "spark is very interested")
                                               .flatMap(x => x.split(" "))
       val list2=List("I am so pround of spark",
                     "spark is very interested")
                                               .map(x => x.split(" "))
      println(list1)
       println(list2)
  }
}
```
