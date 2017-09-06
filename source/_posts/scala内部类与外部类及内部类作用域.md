---
title: Scala内部类与外部类及内部类作用域
date: 2017-05-10 00:54:59
tags: Scala
toc: true
categories: 大数据技术
---
scala内部类与外部类，以及扩大内部类作用域的两种方式，内部类访问外部类的变量的方式。
## 内部类与外部类实例 ##
```
package com.scala.base

import scala.collection.mutable.ArrayBuffer

object Class {
  def main(args: Array[String]): Unit = {
    val c1=new Class
    val leo=c1.register("leo")
    c1.students+=leo

    val c2=new Class
    val jack=c2.register("jack")
    c2.students+=jack
  }
}

class Class{
  class Student(val name:String)
  val students=new ArrayBuffer[Student]
  def register(name:String):Student={
      new Student(name)  
  }
}
```
<!-- more -->
## 通过伴生对象扩大内部类的作用域 ##
```
package com.scala.base

import scala.collection.mutable.ArrayBuffer
/**
 * 通过伴生对象扩大内部类的作用域
 */
object Class1 {
  class Student(val name:String)
  def main(args: Array[String]): Unit = {
    val c1=new Class1
    val leo=c1.register("leo")
    c1.students+=leo

    val c2=new Class1
    val jack=c2.register("jack")
    c1.students+=jack
  }
}

class Class1{
  val students=new ArrayBuffer[Class1.Student]
  def register(name:String):Class1.Student={
      new Class1.Student(name)  
  }
}
```
## 通过类型投影扩大内部类作用域 ##
```
package com.scala.base

import scala.collection.mutable.ArrayBuffer
/**
 * 通过类型投影扩大内部类作用域
 */
object Class2 {
  def main(args: Array[String]): Unit = {
    val c1=new Class2
    val leo=c1.register("leo")
    c1.students+=leo

    val c2=new Class2
    val jack=c2.register("jack")
    c1.students+=jack
  }
}

class Class2{
  class Student(val name:String)
  val students=new ArrayBuffer[Class2#Student]
  def register(name:String):Student={
      new Student(name)  
  }
}
```
## 内部类引用外部类的变量 ##
```
package com.scala.base
/**
 * 内部类引用外部类的变量
 */
object InnerReferOuterClass {
  def  main(args: Array[String]): Unit = {
    val c1=new Class3("c1")
    val leo=c1.register("leo")
    println(leo.introduceMyself)
  }
}
class Class3(val name:String){outer=>
  class Student(val name:String){
    def introduceMyself="hello I am " + name + "I am very happy to join class " + outer.name
  }
  def register(name:String)={
    new Student(name)
  }
}
```
