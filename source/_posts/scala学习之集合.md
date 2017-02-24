---
title: scala学习之集合
date: 2017-02-01 00:54:59
tags: scala，基础
categories: scala
---
1.在scala中定义类是用关键字class
2.可以用new ClassName的方式构建出类的对象
3.如果名称相同，则Object中的内容都是Class的静态内容，也就是说Object中的内容class都可以在没有实例化的时候直接调用可以使用Object中特定方法apply来创建实例。
4.Object中的apply方法是class对象生成的工厂方法，用于控制对象的生成
5.很多框架的代码一直调用抽象类的object的apply方法生成类的实例。抽象类是不能被实例化的，apply调用的是抽象类的子类实现类的实例化。
6.objec 是class的伴生对象，class可以直接访问Object中的一切内容。class是objec的伴生类。Object可以直接访问class的一切内容。
7.在定义class时，可以直接在类后面加入构造参数，此时在apply中也必须有相应的参数。
8.scala中可以在objec中定义多个apply方法。
<!--more -->
```
class HelloOOP(age:Int){
  var name = "spark"
  def sayHello = println("Hi,My name is " + name + " and age is " + age)
}

object HelloOOP {
  var number =0
  def main(args: Array[String]): Unit = {
    println("hello scala oop")
    //val helloOOP = new HelloOOP() //构建对象方式一
    //val helloOOP = HelloOOP() //构建对象方式二
   // val helloOOP01 = HelloOOP() //构建对象方式二
   // val helloOOP02 = HelloOOP() //构建对象方式二
    val helloOOP = HelloOOP(30)
    helloOOP.sayHello
  }

  def apply(age:Int):HelloOOP={
    println("my number is : " + number)
    number += 1
    new HelloOOP(age)
  }  
}
```
