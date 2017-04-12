---
title: scala基础学习-函数式编程
date: 2017-02-09 23:54:59
tags: 函数式编程
toc: true
categories: scala
---
1.函数可以直接赋值给变量
2.函数更常用的方式是匿名函数，定义的时候只需要说明输入参数的类型和函数体，如果要使用可以赋值给一个变量(其实是val常量)
3.函数可以作为参数直接传递给函数，简化了编程的语法
4.函数的返回值也可以是函数,当函数的返回类型是函数时，这个时候就表明了scala的函数实现了闭包
5.scala闭包的内幕，scala函数的背后是类和对象，所以scala的参数都作为对象的成员，所以后续可以继续访问。
6.Curring，复杂的函数编程中经常用到，可以维护变量在内存中的状态，且可以实现返回函数的链式功能，可以实现非常复杂的算法，
<!-- more -->
```
object HelloFunctionProgramming {
  def main(args: Array[String]): Unit = {
    //函数赋值给变量
    val hiData=bigData _
    hiData("Spark")

    //_* 每一个元素
    Array(1 to 10 : _*).map((item:Int)=>2*item).foreach(x => println(x))
    def result=(name:String) => println("Hi " + name)
    result("Java")

    def result1(message:String)=(name:String) => println(message+"  " + name)
    result1("hello")("Java")//currying 函数写法
  }
  //匿名函数
  val f=(name:String) => println("Hi " + name)
  f("kafaka")

  getName(f,"scala")
  def bigData(name:String){
    println("Hi " + name)
  }

  def getName(func:(String) => Unit,name : String){
    func(name)
  }
}
```
